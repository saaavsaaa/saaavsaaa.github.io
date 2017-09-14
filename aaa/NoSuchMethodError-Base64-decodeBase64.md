
有台服务器应用里报错，可稳定重现，但只能在这一台机器上重现：
-----
![Image](/ppp/NoSuchMethodError.png)
-----
先对比了一下，正常和异常的服务器，环境完全一样，其实都没必要对比，因为都是同样配置出来的虚机，连各种小版本都完全一样。
-----
Jhat查看了一下dump：
=====
这是正常的：
------
![Image](/ppp/213Base64.png)
------
![Image](/ppp/213staticdata.png)
-----
这是出错的：
-----
![Image](/ppp/214Base64.png)
-----
![Image](/ppp/214staticdata.png)
-----
这些静态成员有些意思，在正常的环境中是没有这些的，根据这些静态成员，从线上的代码中发现了一个类：
-----
```markdown
package com.xxx.xxx.xxx.xxx.reapel.client.utils;
import java.io.UnsupportedEncodingException;
public class Base64  {
    . . .
    public static byte[] encodeBase64(byte[] binaryData) {
            . . .
    }
    . . .
}
```
-----
这个类里的静态成员和上面截图的完全对的上，这个方法和报错也对的上，除了包不一样，其他的都更像是自定义的这个，回头再看报错处的代码：
-----
![Image](/ppp/base64invoke.png)
-----
把dump下载下来用Jvisualvm查看
=====
正常的：
-----
![Image](/ppp/213instance.png)
-----
有错的：
-----
![Image](/ppp/214instance.png)
=====
先怀疑包的问题，然而用有错服务器上的包直接部署到其他机器上没有问题，拉下来解压看了，都不缺，也都一样
-----
我还在异常的服务器上javap -c RSA了一下，很正常和正常服务器上的完全一样：
-----
```markdown
  #33 = Methodref          #158.#159     // org/apache/commons/codec/binary/Base64.decodeBase64:(Ljava/lang/String;)[B
  #105 = Utf8               (Ljava/lang/String;)[B
  #158 = Class              #191          // org/apache/commons/codec/binary/Base64
  #159 = NameAndType        #192:#105     // decodeBase64:(Ljava/lang/String;)[B
  #191 = Utf8               org/apache/commons/codec/binary/Base64
  #192 = Utf8               decodeBase64

  public static byte[] decryptBASE64(java.lang.String) throws java.lang.Exception;
    Code:
       0: aload_0
       1: invokestatic  #33                 // Method org/apache/commons/codec/binary/Base64.decodeBase64:(Ljava/lang/String;)[B
       4: areturn
```
然后再![Image](/ppp/javapbase64214.png):
-----
```markdown
  public static byte[] decodeBase64(java.lang.String);
    Code:
       0: new           #45                 // class org/apache/commons/codec/binary/Base64
       3: dup
       4: invokespecial #52                 // Method "<init>":()V
       7: aload_0
       8: invokevirtual #53                 // Method decode:(Ljava/lang/String;)[B
      11: areturn
```
-----
其实之前主要是怀疑，加载类的链接过程出错了。由于都是静态方法，据说位置是不会改变的。#32对应的位置本该存在的方法被冒名顶替了？反正已经都找到这了，先接着往下看，如果还想不明白再去研究下链接的部分。于是，顺着就找到了hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp，下面这段应该是静态方法执行的代码，大概：
```markdown
      CASE(_invokestatic): {
        u2 index = Bytes::get_native_u2(pc+1);

        ConstantPoolCacheEntry* cache = cp->entry_at(index);
        // QQQ Need to make this as inlined as possible. Probably need to split all the bytecode cases
        // out so c++ compiler has a chance for constant prop to fold everything possible away.

        if (!cache->is_resolved((Bytecodes::Code)opcode)) {
          CALL_VM(InterpreterRuntime::resolve_invoke(THREAD, (Bytecodes::Code)opcode),
                  handle_exception);
          cache = cp->entry_at(index);
        }

        istate->set_msg(call_method);
        {
          Method* callee;
          if ((Bytecodes::Code)opcode == Bytecodes::_invokevirtual) {
            CHECK_NULL(STACK_OBJECT(-(cache->parameter_size())));
            if (cache->is_vfinal()) {
              callee = cache->f2_as_vfinal_method();
              // Profile final call.
              BI_PROFILE_UPDATE_FINALCALL();
            } else {
              // get receiver
              int parms = cache->parameter_size();
              // this works but needs a resourcemark and seems to create a vtable on every call:
              // Method* callee = rcvr->klass()->vtable()->method_at(cache->f2_as_index());
              //
              // this fails with an assert
              // InstanceKlass* rcvrKlass = InstanceKlass::cast(STACK_OBJECT(-parms)->klass());
              // but this works
              oop rcvr = STACK_OBJECT(-parms);
              VERIFY_OOP(rcvr);
              InstanceKlass* rcvrKlass = (InstanceKlass*)rcvr->klass();
              /*
                Executing this code in java.lang.String:
                    public String(char value[]) {
                          this.count = value.length;
                          this.value = (char[])value.clone();
                     }

                 a find on rcvr->klass() reports:
                 {type array char}{type array class}
                  - klass: {other class}

                  but using InstanceKlass::cast(STACK_OBJECT(-parms)->klass()) causes in assertion failure
                  because rcvr->klass()->oop_is_instance() == 0
                  However it seems to have a vtable in the right location. Huh?

              */
              callee = (Method*) rcvrKlass->start_of_vtable()[ cache->f2_as_index()];
              // Profile virtual call.
              BI_PROFILE_UPDATE_VIRTUALCALL(rcvr->klass());
            }
          } else {
            if ((Bytecodes::Code)opcode == Bytecodes::_invokespecial) {
              CHECK_NULL(STACK_OBJECT(-(cache->parameter_size())));
            }
            callee = cache->f1_as_method();

            // Profile call.
            BI_PROFILE_UPDATE_CALL();
          }

          istate->set_callee(callee);
          istate->set_callee_entry_point(callee->from_interpreted_entry());
#ifdef VM_JVMTI
          if (JvmtiExport::can_post_interpreter_events() && THREAD->is_interp_only_mode()) {
            istate->set_callee_entry_point(callee->interpreter_entry());
          }
#endif /* VM_JVMTI */
          istate->set_bcp_advance(3);
          UPDATE_PC_AND_RETURN(0); // I'll be back...
        }
      }
```
