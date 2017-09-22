-----
    /jdk/launcher/java.c
-----
    JavaMain(void * _args):
    mainClass = LoadMainClass(env, mode, what);
    appClass = GetApplicationClass(env);
    
    
    LoadMainClass(JNIEnv *env, int mode, char *name)
-----

-----

-----
    /share/vm/classfile/classLoader.cpp
-----
    instanceKlassHandle ClassLoader::load_classfile(Symbol* h_name, TRAPS) : 
    
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



-----
    /share/vm/classfile/systemDictionary.cpp
-----

       ClassLoaderData* loader_data = class_loader_data(class_loader);
       unsigned int d_hash = dictionary()->compute_hash(child_name, loader_data); 
###    int d_index = dictionary()->hash_to_index(d_hash);
      
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
      /home/aaa/Github/hotspot/src/share/vm/utilities/hashtable.cpp : 
      #include "utilities/hashtable.inline.hpp"
      /share/vm/utilities/hashtable.inline.hpp :
      // The following method is MT-safe and may be used with caution.
      template <MEMFLAGS F> inline BasicHashtableEntry<F>* BasicHashtable<F>::bucket(int i) {
        return _buckets[i].get_entry();
      }
-----










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
=====

        Klass* check = find_class(d_index, d_hash, name, loader_data);
        if (check != NULL) {
        // Klass is already loaded, so return it after checking/adding protection domain
          k = instanceKlassHandle(THREAD, check);
          class_has_been_loaded = true;
        }
      }
    }

    // must throw error outside of owning lock
    if (throw_circularity_error) {
      assert(!HAS_PENDING_EXCEPTION && load_instance_added == false,"circularity error cleanup");
      ResourceMark rm(THREAD);
      THROW_MSG_NULL(vmSymbols::java_lang_ClassCircularityError(), name->as_C_string());
    }

    if (!class_has_been_loaded) {

      // Do actual loading
      k = load_instance_class(name, class_loader, THREAD);
-----
    k = ClassLoader::load_classfile(class_name, CHECK_(nh));
-----
    /share/vm/classfile/classLoader.cpp
-----
    instanceKlassHandle ClassLoader::load_classfile(Symbol* h_name, TRAPS) 
-----
    instanceKlassHandle result = parser.parseClassFile(h_name,
                                                       loader_data,
                                                       protection_domain,
                                                       parsed_name,
                                                       context.should_verify(classpath_index),
                                                       THREAD);

-----
    --> /share/vm/classfile/classFileParser.cpp 3700


