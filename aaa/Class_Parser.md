#define JAVA_MIN_SUPPORTED_VERSION        45
#define JAVA_MAX_SUPPORTED_VERSION        52
#define JAVA_MAX_SUPPORTED_MINOR_VERSION  0

// Used for two backward compatibility reasons:
// - to check for new additions to the class file format in JDK1.5
// - to check for bug fixes in the format checker in JDK1.5
#define JAVA_1_5_VERSION                  49

// Used for backward compatibility reasons:
// - to check for javac bug fixes that happened after 1.5
// - also used as the max version when running in jdk6
#define JAVA_6_VERSION                    50

// Used for backward compatibility reasons:
// - to check NameAndType_info signatures more aggressively
#define JAVA_7_VERSION                    51

// Extension method support.
#define JAVA_8_VERSION                    52
-----
    /share/vm/classfile/systemDictionary.cpp
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


