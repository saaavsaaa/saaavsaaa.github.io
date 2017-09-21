

-----
    /share/vm/classfile/systemDictionary.cpp
-----
  ???  instanceKlassHandle SystemDictionary::handle_parallel_super_load(
-----
     Klass* SystemDictionary::resolve_instance_class_or_null(Symbol* name,
                                                            Handle class_loader,
                                                            Handle protection_domain,
                                                            TRAPS) {
      assert(name != NULL && !FieldType::is_array(name) &&
             !FieldType::is_obj(name), "invalid class name");

      Ticks class_load_start_time = Ticks::now();

      // UseNewReflection
      // Fix for 4474172; see evaluation for more details
      class_loader = Handle(THREAD, java_lang_ClassLoader::non_reflection_class_loader(class_loader()));
      ClassLoaderData *loader_data = register_loader(class_loader, CHECK_NULL);

      // Do lookup to see if class already exist and the protection domain
      // has the right access
      // This call uses find which checks protection domain already matches
      // All subsequent calls use find_class, and set has_loaded_class so that
      // before we return a result we call out to java to check for valid protection domain
      // to allow returning the Klass* and add it to the pd_set if it is valid
      unsigned int d_hash = dictionary()->compute_hash(name, loader_data);
      
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


