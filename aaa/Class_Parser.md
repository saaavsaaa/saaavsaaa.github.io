
    在[之前排查的一个问题](https://saaavsaaa.github.io/aaa/NoSuchMethodError-Base64-decodeBase64.html) 的结尾还留了一个问题，为什么有的机器会加载正确的类，有的就是错的。因为这一段在上线一个项目，灰度公测阶段，所以拖了些天，穿插着看了看加载相关的一些jvm代码，以前也没看过，就边猜测边看了。
    主要猜测，一是在问题的那篇中打印的load jar处，这里应该会根据已经扫过的jar的路径去加载；另外一个是启动初始化的地方会扫jar;顺便也看了些相关的代码，这里算是代码的记录吧，因为还没有证明，问题也没有解决。

-----

    我找到的可能和jar包相关的，然而并不是我想要的部分，这里应该是在加载JDK相关的jar
    
-----
    /jdk/launcher/java.c
-----
    int JLI_Launch(int argc, char ** argv, .... :

    static void SelectVersion(int argc, char **argv, char **main_class):
    if (splash_file_name && !headlessflag) {
        char* splash_file_entry = JLI_MemAlloc(JLI_StrLen(SPLASH_FILE_ENV_ENTRY "=")+JLI_StrLen(splash_file_name)+1);
        JLI_StrCpy(splash_file_entry, SPLASH_FILE_ENV_ENTRY "=");
        JLI_StrCat(splash_file_entry, splash_file_name);
        putenv(splash_file_entry);
    }
    if (splash_jar_name && !headlessflag) {
        char* splash_jar_entry = JLI_MemAlloc(JLI_StrLen(SPLASH_JAR_ENV_ENTRY "=")+JLI_StrLen(splash_jar_name)+1);
        JLI_StrCpy(splash_jar_entry, SPLASH_JAR_ENV_ENTRY "=");
        JLI_StrCat(splash_jar_entry, splash_jar_name);
        putenv(splash_jar_entry);
    }
-----

    引用它的地方是：

-----
    /launcher/main.c :
    int main(int argc, char **argv):
     ...
        return JLI_Launch(margc, margv,
                   sizeof(const_jargs) / sizeof(char *), const_jargs,
                   sizeof(const_appclasspath) / sizeof(char *), const_appclasspath,
                   FULL_VERSION,
                   DOT_VERSION,
                   (const_progname != NULL) ? const_progname : *margv,
                   (const_launcher != NULL) ? const_launcher : *margv,
                   (const_jargs != NULL) ? JNI_TRUE : JNI_FALSE,
                   const_cpwildcard, const_javaw, const_ergo_class);
-----
   
    这里应该是应用启动的地方，这里还在jdk中：
    
-----
    JavaMain(void * _args):
    mainClass = LoadMainClass(env, mode, what);
    appClass = GetApplicationClass(env);
    
    
    LoadMainClass(JNIEnv *env, int mode, char *name)
    
        NULL_CHECK0(mid = (*env)->GetStaticMethodID(env, cls,
                "checkAndLoadMain",
                "(ZILjava/lang/String;)Ljava/lang/Class;"));

    (*env)->CallStaticVoidMethod(env, mainClass, mainID, mainArgs);
                
-----

    加载了启动类，根据参数类型什么的检查是不是启动类，然后调用，调用部分就到jvm里了，注意JNIEnv。

-----
    /share/vm/prims/jni.h:
    typedef const struct JNINativeInterface_ *JNIEnv;
    
    struct JNIEnv_ {
    const struct JNINativeInterface_ *functions;
    #ifdef __cplusplus

    jint GetVersion() {
        return functions->GetVersion(this);
    }
    jclass DefineClass(const char *name, jobject loader, const jbyte *buf,
                       jsize len) {
        return functions->DefineClass(this, name, loader, buf, len);
    }
    jclass FindClass(const char *name) {
        return functions->FindClass(this, name);
    }
    . . .
    struct JNINativeInterface_ {
    void *reserved0;
    void *reserved1;
    void *reserved2;

    void *reserved3;
    jint (JNICALL *GetVersion)(JNIEnv *env);

    jclass (JNICALL *DefineClass)
      (JNIEnv *env, const char *name, jobject loader, const jbyte *buf,
       jsize len);
    jclass (JNICALL *FindClass)
      (JNIEnv *env, const char *name);

    
    JNI_ENTRY(jmethodID, jni_GetStaticMethodID(JNIEnv *env, jclass clazz,
          const char *name, const char *sig))
        JNIWrapper("GetStaticMethodID");

    JNI_ENTRY(void, jni_CallStaticVoidMethod(JNIEnv *env, jclass cls, jmethodID methodID, ...))
      JNIWrapper("CallStaticVoidMethod");

    #define JNI_ENTRY(result_type, header)                               \
        JNI_ENTRY_NO_PRESERVE(result_type, header)                       \
        WeakPreserveExceptionMark __wem(thread);

    #define JNI_ENTRY_NO_PRESERVE(result_type, header)             \
    extern "C" {                                                         \
      result_type JNICALL header {                                \
        JavaThread* thread=JavaThread::thread_from_jni_environment(env); \
        assert( !VerifyJNIEnvThread || (thread == Thread::current()), "JNIEnv is only valid in same thread"); \
        ThreadInVMfromNative __tiv(thread);                              \
        debug_only(VMNativeEntryWrapper __vew;)                          \
        VM_ENTRY_BASE(result_type, header, thread)
-----

    上面那条线估计找不到什么了，从启动开始/share/vm/runtime/thread.cpp ：
    
-----    

    Threads::create_vm(JavaVMInitArgs* args, bool* canTryAgain)：
    jint init_globals() :
      HandleMark hm;
      management_init();
      bytecodes_init();
      classLoader_init();
      codeCache_init();
      VM_Version_init();
      os_init_globals();
      stubRoutines_init1();
    jint status = universe_init();  // dependent on codeCache_init and
                                    // stubRoutines_init1 and metaspace_init.
    . . .
    
-----
    /home/aaa/Github/hotspot/src/share/vm/services/management.cpp:
    management_init()

-----

    这里启动了写有意思的东西，问题查完了需要看看，比如safepoint、PerfData什么的一些计数器，这里有ClassLoadingService先翻翻，有class的load和unload的通知什么的。

-----
    --> /share/vm/runtime/init.cpp : jint init_globals()  // call constructors at startup (main Java thread)
    
    /share/vm/memory/universe.cpp
    jint universe_init()
    
-----
    /share/vm/memory/metaspaceShared.cpp的void MetaspaceShared::initialize_shared_spaces()方法中调用的。
    initialize_shared_spaces方法中的注释：
    Create the package info table using the bucket array at this spot in
    the misc data space.  Since the package info table is never
    modified, this region (of mapped pages) will be (effectively, if
    not explicitly) read-only.
    The following data in the shared misc data region are the linked
    list elements (HashtableEntry objects) for the symbol table, string
    table, and shared dictionary.  The heap objects refered to by the
    symbol table, string table, and shared dictionary are permanent and
    unmovable.  Since new entries added to the string and symbol tables
     are always added at the beginning of the linked lists, THESE LINKED
    LIST ELEMENTS ARE READ-ONLY.
    从注释来看，这次似乎接近了。
    
    if (DumpSharedSpaces) {
      MetaspaceShared::prepare_for_dumping();
    }
    
    void MetaspaceShared::prepare_for_dumping() {
      ClassLoader::initialize_shared_path();
      FileMapInfo::allocate_classpath_entry_table();
    }
    
    /share/vm/memory/filemap.hpp : 
    void FileMapInfo::allocate_classpath_entry_table() :
    
-----







-----

    下面这个都认识：

-----
    /share/vm/classfile/classLoader.cpp
-----
    PackageInfo* ClassLoader::lookup_package(const char *pkgname) {
      const char *cp = strrchr(pkgname, '/');
      if (cp != NULL) {
        // Package prefix found
        int n = cp - pkgname + 1;
        return _package_hash_table->get_entry(pkgname, n);
      }
      return NULL;
    }
-----
    instanceKlassHandle record_result(const int classpath_index,
                                          ClassPathEntry* e, instanceKlassHandle result, TRAPS) {
          if (ClassLoader::add_package(_file_name, classpath_index, THREAD)) {
-----

    顺着这个方法的引用找到了：

-----
    void ClassLoader::create_package_info_table(HashtableBucket<mtClass> *t, int length,
                                                int number_of_entries) {
      assert(_package_hash_table == NULL, "One package info table allowed.");
      assert(length == package_hash_table_size * sizeof(HashtableBucket<mtClass>),
             "bad shared package info size.");
      _package_hash_table = new PackageHashtable(package_hash_table_size, t,
                                                 number_of_entries);
    }
-----
 


    











-----
     instanceKlassHandle ClassLoader::load_classfile(Symbol* h_name, TRAPS) : 

     ClassPathEntry* e = NULL;
     #e = _first_entry;
-----   
    _first_entry来源于：
-----
    void ClassLoader::add_to_list(ClassPathEntry *new_entry) {
      if (new_entry != NULL) {
        if (_last_entry == NULL) {
          _first_entry = _last_entry = new_entry;
        } else {
          _last_entry->set_next(new_entry);
          _last_entry = new_entry;
        }
      }
      _num_entries ++;
    }
-----   





    void ClassLoader::setup_search_path(const char *class_path, bool canonicalize) {
      int offset = 0;
      int len = (int)strlen(class_path);
      int end = 0;

      // Iterate over class path entries
      for (int start = 0; start < len; start = end) {
        while (class_path[end] && class_path[end] != os::path_separator()[0]) {
          end++;
        }
        EXCEPTION_MARK;
        ResourceMark rm(THREAD);
        char* path = NEW_RESOURCE_ARRAY(char, end - start + 1);
        strncpy(path, &class_path[start], end - start);
        path[end - start] = '\0';
        if (canonicalize) {
          char* canonical_path = NEW_RESOURCE_ARRAY(char, JVM_MAXPATHLEN + 1);
          if (get_canonical_path(path, canonical_path, JVM_MAXPATHLEN)) {
            path = canonical_path;
          }
        }
    #    update_class_path_entry_list(path, /*check_for_duplicates=*/canonicalize);
    #if INCLUDE_CDS
        if (DumpSharedSpaces) {
          check_shared_classpath(path);
        }
    #endif
        while (class_path[end] == os::path_separator()[0]) {
          end++;
        }
      }
    }





-----
    ClassFileParser parser(stream);
    ClassLoaderData* loader_data = ClassLoaderData::the_null_class_loader_data();
    Handle protection_domain;
    TempNewSymbol parsed_name = NULL;
    instanceKlassHandle result = parser.parseClassFile(h_name,
                                                       loader_data,
                                                       protection_domain,
                                                       parsed_name,
                                                       context.should_verify(classpath_index),
                                                       THREAD);
-----
    instanceKlassHandle result = parser.parseClassFile(h_name,
                                                       loader_data,
                                                       protection_domain,
                                                       parsed_name,
                                                       context.should_verify(classpath_index),
                                                       THREAD);

-----
    /share/vm/classfile/classFileParser.cpp 3700
-----

    上面加载类文件，从流中读取，流是classloader中传给ClassFileParser构造函数的。下面从dictionary中查找，字典结构，先hash出桶index，然后根据index查找：

-----
    /share/vm/classfile/systemDictionary.cpp
-----

      ClassLoaderData* loader_data = class_loader_data(class_loader);
      unsigned int d_hash = dictionary()->compute_hash(child_name, loader_data); 
      int d_index = dictionary()->hash_to_index(d_hash);
      
      Klass* probe = dictionary()->find(d_index, d_hash, name, loader_data,
                                          protection_domain, THREAD);
-----
    Klass* check = find_class(d_index, d_hash, name, loader_data)
-----
    Klass* SystemDictionary::find_class(int index, unsigned int hash,
                                          Symbol* class_name,
                                          ClassLoaderData* loader_data) {
      assert_locked_or_safepoint(SystemDictionary_lock);
      assert (index == dictionary()->index_for(class_name, loader_data),
              "incorrect index?");

      Klass* k = dictionary()->find_class(index, hash, class_name, loader_data);
      return k;
    }
-----
    /share/vm/classfile/dictionary.cpp
-----
    Klass* Dictionary::find_class(int index, unsigned int hash,
                                    Symbol* name, ClassLoaderData* loader_data) {
      assert_locked_or_safepoint(SystemDictionary_lock);
      assert (index == index_for(name, loader_data), "incorrect index?");

      DictionaryEntry* entry = get_entry(index, hash, name, loader_data);
      return (entry != NULL) ? entry->klass() : (Klass*)NULL;
    }
-----
    DictionaryEntry* Dictionary::get_entry(int index, unsigned int hash,
                                           Symbol* class_name,
                                           ClassLoaderData* loader_data) {
      debug_only(_lookup_count++);
      for (DictionaryEntry* entry = bucket(index);
                            entry != NULL;
                            entry = entry->next()) {
        if (entry->hash() == hash && entry->equals(class_name, loader_data)) {
          return entry;
        }
        debug_only(_lookup_length++);
      }
      return NULL;
    }
-----
      DictionaryEntry* bucket(int i) {
        return (DictionaryEntry*)Hashtable<Klass*, mtClass>::bucket(i);
      }
-----

    这个词典的数据结构扩展至hashtable，因为对外提供的是内联模板的方法，所以单独放在hashtable.inline，模板感觉就相当于泛型：

-----
      /hotspot/src/share/vm/utilities/hashtable.cpp : 
      #include "utilities/hashtable.inline.hpp"
      /share/vm/utilities/hashtable.inline.hpp :
      // The following method is MT-safe and may be used with caution.
      template <MEMFLAGS F> inline BasicHashtableEntry<F>* BasicHashtable<F>::bucket(int i) {
        return _buckets[i].get_entry();
      }
-----

    上面查出来的东西，应该...不太确定，是这里加进去的，常量池的例子，开头那个问题里有：

-----
    /share/vm/oops/constantPool.cpp
    void SymbolHashMap::add_entry(Symbol* sym, u2 value) {
      char *str = sym->as_utf8();
      unsigned int hash = compute_hash(str, sym->utf8_length());
      unsigned int index = hash % table_size();

      // check if already in map
      // we prefer the first entry since it is more likely to be what was used in
      // the class file
      for (SymbolHashMapEntry *en = bucket(index); en != NULL; en = en->next()) {
        assert(en->symbol() != NULL, "SymbolHashMapEntry symbol is NULL");
        if (en->hash() == hash && en->symbol() == sym) {
            return;  // already there
        }
      }

      SymbolHashMapEntry* entry = new SymbolHashMapEntry(hash, sym, value);
      entry->set_next(bucket(index));
      _buckets[index].set_entry(entry);
      assert(entry->symbol() != NULL, "SymbolHashMapEntry symbol is NULL");
    }
-----

