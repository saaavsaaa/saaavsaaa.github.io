
    在[之前排查的一个问题](https://saaavsaaa.github.io/aaa/NoSuchMethodError-Base64-decodeBase64.html) 的结尾还留了一个问题，为什么有的机器会加载正确的类，有的就是错的。因为这一段在上线一个项目，灰度公测阶段，所以拖了些天，穿插着看了看加载相关的一些hotspot代码，以前也没看过，就边猜测边看了。因为是穿插着看，怕今天看的明天就忘了，所以下面流水账记录了我看的过程。
    
    看了一些jvm代码（下面有看的过程中的记录，有兴趣的可以看看），最终在rt.jar中找到了JarFile这个类，代码在jdk的src.zip包里的。
    用 https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/report/btrace/JarBTrace.java 脚本跟踪了一下：
    
-----    

    正常的服务器脚本的输出：
    
![Image](/ppp/213jarload.png)
    
-----    
    
    异常的服务器上脚本输出：
    
![Image](/ppp/214jarload.png)
    
-----

    这个问题在多台服务器上都出现了，所以搜集多台的输出后可以确认这个规律，其实在下面代码的记录中也可以看出，如果有多个同名的类只会加载其中第一个。在不出问题的服务器上commons-codec-1.10.jar是在http-1.1.0.jar之前加载的，而出问题的服务器正好相反。

-----

    /usr/lib/jvm/java-8-oracle/jre/lib/rt.jar!/sun/misc/URLClassPath.class中的private ArrayList<URL> path有所有加载的jar的URL。这个类里还有每个jar的loader：ArrayList<URLClassPath.Loader> loaders，jar路径和jarLoader的映射：HashMap<String, URLClassPath.Loader> lmap。lmap通过private synchronized URLClassPath.Loader getLoader(int var1)方法从Stack<URL> urls中弹出的jar来构造loader并加入映射的。有两个URLClassLoader加载不同的URLClassPath实例，一个只有jaccess.jar等几个，一个有所有的。不过这都不重要，这些jar都是从classpath加载的。于是我在几个服务器上用jinfo输出了java.class.path，并对比了一下发现正常服务器和服务器中，这两个jar的顺序果然是不一样的。那么我估计问题是出现jvm对java.class.path赋值的前面了，或许是操作系统什么的。
    
    于是决定看看java.class.path初始化的情况，代码在/home/aaa/Github/hotspot/src/share/vm/runtime/arguments.cpp:_java_class_path = new SystemProperty("java.class.path", "",  true)。SystemProperty的构造在/home/aaa/Github/hotspot/src/share/vm/runtime/arguments.hpp：

-----
     // Constructor
     SystemProperty(const char* key, const char* value, bool writeable) {
       if (key == NULL) {
         _key = NULL;
       } else {
         _key = AllocateHeap(strlen(key)+1, mtInternal);
         strcpy(_key, key);
       }
       if (value == NULL) {
         _value = NULL;
       } else {
         _value = AllocateHeap(strlen(value)+1, mtInternal);
         ba(_value, value);
       }
       _next = NULL;
       _writeable = writeable;
     }
    };
-----

    构造里似乎没什么关系，然后回到cpp仔细看了下，发现：
    
```markdown
    // following are JVMTI agent writeable properties.
    // Properties values are set to NULL and they are
    // os specific they are initialized in os::init_system_properties_values().
     
    // Set OS specific system properties values
    os::init_system_properties_values();
```

    这个方法的定义在os.hpp中，不过实现是不同的系统不一样，所以并没有在对应的cpp中，我是linux系统，所以我去找了 /home/aaa/Github/hotspot/src/os/linux/vm/os_linux.cpp，不过也没有我想要的。arguments.cpp中只有对endorsed，ext，bootclasspath的目录文件读取，看来配置的classpath不在这里。
    
-----

    

-----






/home/aaa/Github/hotspot/src/share/vm/runtime/init.cpp
jint init_globals()

/home/aaa/Github/hotspot/src/share/vm/classfile/classLoader.cpp
void classLoader_init() {
  ClassLoader::initialize();
}

void ClassLoader::initialize()


void ClassLoader::setup_bootstrap_search_path()


-XX:+TraceClassPaths
[Bootstrap loader class path=/usr/lib/jvm/java-8-oracle/jre/lib/resources.jar:/usr/lib/jvm/java-8-oracle/jre/lib/rt.jar:/usr/lib/jvm/java-8-oracle/jre/lib/sunrsasign.jar:/usr/lib/jvm/java-8-oracle/jre/lib/jsse.jar:/usr/lib/jvm/java-8-oracle/jre/lib/jce.jar:/usr/lib/jvm/java-8-oracle/jre/lib/charsets.jar:/usr/lib/jvm/java-8-oracle/jre/lib/jfr.jar:/usr/lib/jvm/java-8-oracle/jre/classes]

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
    update_class_path_entry_list(path, /*check_for_duplicates=*/canonicalize);
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

    下面是找线索的时候胡乱看的一些代码的记录，从启动部分开始找加载：

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
    jint init_globals() : // call constructors at startup (main Java thread)
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
    这里启动了写有意思的东西，问题查完了需要看看，比如safepoint、PerfData什么的一些计数器，这里有ClassLoadingService先翻翻，有class的load和unload的通知什么的。
    
    Bytecodes::initialize()，看样子所有的jvm指令都在这了。

    classLoader_init:
       ClassLoader::initialize():
          ClassLoader::setup_bootstrap_search_path():
               _shared_paths_misc_info->add_boot_classpath(sys_class_path);
               setup_search_path(sys_class_path);
-----
    -->/share/vm/memory/universe.cpp :
    jint universe_init() 
    --> /share/vm/memory/metaspaceShared.cpp:
    void Metaspace::global_initialize() --> void MetaspaceGC::initialize()这部分先标记下，回头看
    
    void MetaspaceShared::initialize_shared_spaces()：
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
    
    从注释来看，这次似乎接近了。而且Metaspace的功能很相关。
    
    if (DumpSharedSpaces) {
      MetaspaceShared::prepare_for_dumping();
    }
    
    void MetaspaceShared::prepare_for_dumping() {
      ClassLoader::initialize_shared_path();
      FileMapInfo::allocate_classpath_entry_table();
    }
    
    /share/vm/memory/filemap.hpp : 
    bool FileMapInfo::initialize()
        init_from_file(_fd);
    void FileMapInfo::allocate_classpath_entry_table() :
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
    ...
    serialize(&rc);
       void MetaspaceShared::serialize(SerializeClosure* soc)
            Universe::serialize(soc, true);
               _finalizer_register_cache->serialize(f);
               _loader_addClass_cache->serialize(f);
               _pd_implies_cache->serialize(f);
               _throw_illegal_access_error_cache->serialize(f);
                     // CDS support.  Replace the klass in this with the archive version
                     // could use this for Enhanced Class Redefinition also.
                     void serialize(SerializeClosure* f) {
                       f->do_ptr((void**)&_klass);
                     }
            // Dump/restore references to commonly used names and signatures.
            vmSymbols::serialize(soc);
-----

    此处担心我翻译不好，还是直接看注释比较好。
    
-----
    MetaspaceShared::prepare_for_dumping():
          ClassLoader::initialize_shared_path();
          FileMapInfo::allocate_classpath_entry_table():
              ClassLoaderData* loader_data = ClassLoaderData::the_null_class_loader_data();
              size_t entry_size = SharedClassUtil::shared_class_path_entry_size();

              for (int pass=0; pass<2; pass++) {
                ClassPathEntry *cpe = ClassLoader::classpath_entry(0);
-----               
               
     看到这，我有些方了，似乎找错地方了，应该去jdk里找找，后面就随便贴一下，反正根据命名和注释都能看出来是干什么。
     
-----     
       interpreter_init();  // before any methods loaded
       invocationCounter_init();  // before any methods loaded
       marksweep_init();
       accessFlags_init();
       templateTable_init();
       InterfaceSupport_init();
       SharedRuntime::generate_stubs();
       universe2_init();  // dependent on codeCache_init and stubRoutines_init1
       referenceProcessor_init();
       jni_handles_init();
     #if INCLUDE_VM_STRUCTS
       vmStructs_init();
     #endif // INCLUDE_VM_STRUCTS

       vtableStubs_init();
       InlineCacheBuffer_init();
       compilerOracle_init();
       compilationPolicy_init();
       compileBroker_init();
       VMRegImpl::set_regName();

       if (!universe_post_init()) {
         return JNI_ERR;
       }
       javaClasses_init();   // must happen after vtable initialization
       stubRoutines_init2(); // note: StubRoutines need 2-phase init

     #if INCLUDE_NMT
       // Solaris stack is walkable only after stubRoutines are set up.
       // On Other platforms, the stack is always walkable.
       NMT_stack_walkable = true;
     #endif // INCLUDE_NMT

       // All the flags that get adjusted by VM_Version_init and os::init_2
       // have been set so dump the flags now.
       if (PrintFlagsFinal) {
         CommandLineFlags::printFlags(tty, false);
       }

       return JNI_OK;
     }
-----

     然后就是向操作系统请求创建线程什么的，还有下面classloader也看了一些
     
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
     instanceKlassHandle ClassLoader::load_classfile(Symbol* h_name, TRAPS) : 

     ClassPathEntry* e = NULL;
     #e = _first_entry;
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

     总体来说多看看还是有好处的，等我把原因找到了再回来接着看，先转向jdk。
     
-----     
