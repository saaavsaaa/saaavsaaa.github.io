    # Problematic frame:
    # V  [libjvm.so+0x68fddb]  java_lang_Class::signers(oopDesc*)+0x1b

  一般当前线程是crash的直接原因，可以看出它就是虚拟机线程，这次崩之前应用记录了out of memory
```markdown
  Current thread (0x00007fa3a00b5800):  VMThread [stack: 0x00007fa390cd7000,0x00007fa390dd8000] [id=19257]
```

  崩的时候线程栈里的操作都是虚拟机的，正在打Dump这是。。。，打Dump打崩了？
```markdown
  Stack: [0x00007fa390cd7000,0x00007fa390dd8000],  sp=0x00007fa390dd6720,  free space=1021k
  Native frames: (J=compiled Java code, j=interpreted, Vv=VM code, C=native code)
  V  [libjvm.so+0x68fddb]  java_lang_Class::signers(oopDesc*)+0x1b
  V  [libjvm.so+0x61c072]  DumperSupport::dump_class_and_array_classes(DumpWriter*, Klass*)+0xb2
  V  [libjvm.so+0x467a86]  ClassLoaderDataGraph::classes_do(void (*)(Klass*))+0x36
  V  [libjvm.so+0x61f4ce]  VM_HeapDumper::doit()+0x23e
  V  [libjvm.so+0xac55d5]  VM_Operation::evaluate()+0x55
  V  [libjvm.so+0xac39aa]  VMThread::evaluate_operation(VM_Operation*)+0xba
  V  [libjvm.so+0xac3d2e]  VMThread::loop()+0x1ce
  V  [libjvm.so+0xac41a0]  VMThread::run()+0x70
  V  [libjvm.so+0x91ef78]  java_start(Thread*)+0x108

  VM_Operation (0x00007fa32b7f66a0): HeapDumper, mode: safepoint, requested by thread 0x00007fa32c039800
```

```markdown
  RAX=0x0000000000000040 is an unknown value
  RBX=0x00007fa32b7f6730 is pointing into the stack for thread: 0x00007fa32c039800
  RCX=0x00007fa30adfd010 is an unknown value
  RDX=0x0000000000000000 is an unknown value
  RSP=0x00007fa390dd6720 is an unknown value
  RBP=0x00007fa390dd6720 is an unknown value
  RSI=0x0000000000000000 is an unknown value
  RDI=0x0000000000000000 is an unknown value
  R8 =0x0000000000000000 is an unknown value
  R9 =0x000000000000002a is an unknown value
  R10=0x0000000000000000 is an unknown value
  R11=0x00007fa3a7b3db20: <offset 0x63db20> in /usr/local/jdk1.8.0_91/jre/lib/amd64/server/libjvm.so at 0x00007fa3a7500000
  R12=0x00007fa3a7b1c420: <offset 0x61c420> in /usr/local/jdk1.8.0_91/jre/lib/amd64/server/libjvm.so at 0x00007fa3a7500000
  R13=0x00000001009e1dc8 is pointing into metadata
  R14=0x00007fa390dd673c is an unknown value
  R15=0x00007fa32b7f66a0 is pointing into the stack for thread: 0x00007fa32c039800


  Other Threads:
  =>0x00007fa3a00b5800 VMThread [stack: 0x00007fa390cd7000,0x00007fa390dd8000] [id=19257]
    0x00007fa3a010b000 WatcherThread [stack: 0x00007fa381eff000,0x00007fa382000000] [id=19266]
```

  在安全点，这说明在GC，同时大量线程都在block也证明了这一点，Stop了The World
```markdown

  VM state:at safepoint (normal execution)

  VM Mutex/Monitor currently owned by a thread:  ([mutex/lock_event])
  [0x00007fa3a0006ac0] Threads_lock - owner thread: 0x00007fa3a00b5800
  [0x00007fa3a0007040] Heap_lock - owner thread: 0x00007fa32c039800

  Heap:
   par new generation   total 306688K, used 306687K [0x00000000c0000000, 0x00000000d4cc0000, 0x00000000d4cc0000)
    eden space 272640K, 100% used [0x00000000c0000000, 0x00000000d0a40000, 0x00000000d0a40000)
    from space 34048K,  99% used [0x00000000d2b80000, 0x00000000d4cbfc20, 0x00000000d4cc0000)
    to   space 34048K,   0% used [0x00000000d0a40000, 0x00000000d0a40000, 0x00000000d2b80000)
   concurrent mark-sweep generation total 707840K, used 707839K [0x00000000d4cc0000, 0x0000000100000000, 0x0000000100000000)
   Metaspace       used 55241K, capacity 60784K, committed 61032K, reserved 1099776K
    class space    used 8809K, capacity 10268K, committed 10404K, reserved 1048576K

  Card table byte_map: [0x00007fa3a60c7000,0x00007fa3a62c8000] byte_map_base: 0x00007fa3a5ac7000

  Marking Bits: (CMSBitMap*) 0x00007fa3a0061848
   Bits: [0x00007fa3a509b000, 0x00007fa3a5b68000)

  Mod Union Table: (CMSBitMap*) 0x00007fa3a0061908
   Bits: [0x00007fa3a506f000, 0x00007fa3a509a340)

  Polling page: 0x00007fa3a8703000

  CodeCache: size=245760Kb used=47105Kb max_used=47139Kb free=198654Kb
   bounds [0x00007fa391000000, 0x00007fa393e70000, 0x00007fa3a0000000]
   total_blobs=11243 nmethods=10606 adapters=550
   compilation: enabled
```


  各种block，没什么看头
```markdown
  identity_map_codec.rb:51" daemon [_thread_blocked, id=19284, stack(0x00007fa32b1f8000,0x00007fa32b3f9000)]
  0x00007fa32c03a800 JavaThread "[main]<file" daemon [_thread_blocked, id=19283, stack(0x00007fa32b3f9000,0x00007fa32b5fa000)]
  0x00007fa32c039800 JavaThread "[main]<file" daemon [_thread_blocked, id=19282, stack(0x00007fa32b5fa000,0x00007fa32b7fb000)]
  0x00007fa32c038000 JavaThread "[main]<file" daemon [_thread_blocked, id=19281, stack(0x00007fa32b7fc000,0x00007fa32b9fd000)]
  0x00007fa32c036800 JavaThread "[main]<file" daemon [_thread_blocked, id=19280, stack(0x00007fa32b9fd000,0x00007fa32bbfe000)]
  0x00007fa32c02b800 JavaThread "[main]<file" daemon [_thread_blocked, id=19279, stack(0x00007fa32bbfe000,0x00007fa32bdff000)]
  0x00007fa32c025800 JavaThread "[main]<file" daemon [_thread_blocked, id=19278, stack(0x00007fa32bdff000,0x00007fa32c000000)]
  0x00007fa32c023000 JavaThread "[main]<file" daemon [_thread_blocked, id=19277, stack(0x00007fa360119000,0x00007fa36031a000)]
  0x00007fa334002800 JavaThread "[main]-pipeline-manager" daemon [_thread_blocked, id=19276, stack(0x00007fa36031a000,0x00007fa36051b000)]
  0x00007fa340001000 JavaThread "pool-2-thread-1" [_thread_blocked, id=19275, stack(0x00007fa36051b000,0x00007fa36071c000)]
```
  
  这应该是说，GC的时候报了内存溢出，溢出的时候要打个Dump，然而IO超时，于是就崩了，就崩了(╯‵□′)╯︵┻━┻
```markdown
  Internal exceptions (10 events):
  Event: 3175189.838 Thread 0x00007fa32c1ee800 Exception <a 'java/net/SocketTimeoutException': Read timed out> (0x00000000c1bde570) thrown at [/HUDSON/workspace/8-2-build-linux-amd64/jdk8u91/6644/hotspot/src/share/vm/prims/jni.cpp, line 735]
  Event: 3175190.227 Thread 0x00007fa32c1f0800 Exception <a 'java/net/SocketTimeoutException': Read timed out> (0x00000000c1e0def8) thrown at [/HUDSON/workspace/8-2-build-linux-amd64/jdk8u91/6644/hotspot/src/share/vm/prims/jni.cpp, line 735]
  Event: 3175205.591 Thread 0x00007fa32c1ee000 Exception <a 'java/net/SocketTimeoutException': Read timed out> (0x00000000c2c114e0) thrown at [/HUDSON/workspace/8-2-build-linux-amd64/jdk8u91/6644/hotspot/src/share/vm/prims/jni.cpp, line 735]
  Event: 3175206.057 Thread 0x00007fa32c1f0800 Exception <a 'java/net/SocketTimeoutException': Read timed out> (0x00000000c32ddfd0) thrown at [/HUDSON/workspace/8-2-build-linux-amd64/jdk8u91/6644/hotspot/src/share/vm/prims/jni.cpp, line 735]
  Event: 3175210.248 Thread 0x00007fa32c1ee000 Exception <a 'java/net/SocketTimeoutException': Read timed out> (0x00000000c013b2b8) thrown at [/HUDSON/workspace/8-2-build-linux-amd64/jdk8u91/6644/hotspot/src/share/vm/prims/jni.cpp, line 735]
  Event: 3175215.785 Thread 0x00007fa32c1f0800 Exception <a 'java/net/SocketTimeoutException': Read timed out> (0x00000000c0664360) thrown at [/HUDSON/workspace/8-2-build-linux-amd64/jdk8u91/6644/hotspot/src/share/vm/prims/jni.cpp, line 735]
  Event: 3175220.940 Thread 0x00007fa340006800 Implicit null exception at 0x00007fa39157ea7f to 0x00007fa39157f395
  Event: 3175221.675 Thread 0x00007fa32c1ee000 Exception <a 'java/net/SocketTimeoutException': Read timed out> (0x00000000d4c8ebf8) thrown at [/HUDSON/workspace/8-2-build-linux-amd64/jdk8u91/6644/hotspot/src/share/vm/prims/jni.cpp, line 735]
  Event: 3175225.052 Thread 0x00007fa32c025800 Implicit null exception at 0x00007fa3922e05f9 to 0x00007fa3922e1fb5
  Event: 3175225.052 Thread 0x00007fa340001000 Implicit null exception at 0x00007fa393615368 to 0x00007fa393615bbd

  Events (10 events):
  Event: 3175226.764 Executing VM operation: GenCollectForAllocation done
  Event: 3175226.764 Executing VM operation: GenCollectForAllocation
  Event: 3175227.434 Executing VM operation: GenCollectForAllocation done
  Event: 3175227.434 Executing VM operation: GenCollectForAllocation
  Event: 3175228.091 Executing VM operation: GenCollectForAllocation done
  Event: 3175228.091 Executing VM operation: GenCollectForAllocation
  Event: 3175228.762 Executing VM operation: GenCollectForAllocation done
  Event: 3175228.762 Executing VM operation: GenCollectForAllocation
  Event: 3175229.256 Executing VM operation: GenCollectForAllocation done
  Event: 3175229.256 Executing VM operation: HeapDumper
```

微信公众号：

![Image](/ppp/20170902204445.jpg)
