  btrace已经修改了这个功能https://github.com/btraceio/btrace/issues/327
  我之前自己改了之后，抽空修了一下，然后给提了个pull request，将脚本中是否包含原方法参数和是否包含异常参数分开处理，有参数就打印参数：
  
-----

                        Type[] sources = Type.getArgumentTypes(getDescriptor());
                        useArgs = sources.length == 0 || om.getMethodParameter() == -1;
    
                        if (useArgs){
                            vr = validateArguments(om, actionArgTypes, new Type[]{THROWABLE_TYPE});
                        } else {
                            vr = validateArguments(om, actionArgTypes, Type.getArgumentTypes(getDescriptor()));
                        }
			
                            ArgumentProvider[] actionArgs;
                            Label l;
                            if (useArgs){
                                actionArgs = buildArgsWithoutParas(throwableIndex);
                                l = levelCheck(om, bcn.getClassName(true));
                                loadArguments(actionArgs);
                            } else {
                                actionArgs = loadArgsWithParas(throwableIndex);
                                l = levelCheck(om, bcn.getClassName(true));
                                loadArguments(vr, actionArgTypes, isStatic(), actionArgs);
                            }
			
		    private ArgumentProvider[] loadArgsWithParas(int throwableIndex){

                        ArgumentProvider[] actionArgs = new ArgumentProvider[5];
                        actionArgs[0] = constArg(throwableIndex, THROWABLE_TYPE);
                        actionArgs[1] = constArg(om.getClassNameParameter(), className.replace('/', '.'));
                        actionArgs[2] = constArg(om.getMethodParameter(), getName(om.isMethodFqn()));
                        actionArgs[3] = selfArg(om.getSelfParameter(), Type.getObjectType(className));
                        actionArgs[4] = new ArgumentProvider(asm, om.getDurationParameter()) {
                            @Override
                            public void doProvide() {
                                MethodTrackingExpander.DURATION.insert(mv);
                            }
                        };
        
                        return actionArgs;
                    }
    
                    private ArgumentProvider[] buildArgsWithoutParas(int throwableIndex){
                        ArgumentProvider[] actionArgs = new ArgumentProvider[5];
        
                        actionArgs[0] = localVarArg(vr.getArgIdx(0), THROWABLE_TYPE, throwableIndex);
                        actionArgs[1] = constArg(om.getClassNameParameter(), className.replace('/', '.'));
                        actionArgs[2] = constArg(om.getMethodParameter(), getName(om.isMethodFqn()));
                        actionArgs[3] = selfArg(om.getSelfParameter(), Type.getObjectType(className));
                        actionArgs[4] = new ArgumentProvider(asm, om.getDurationParameter()) {
                            @Override
                            public void doProvide() {
                                MethodTrackingExpander.DURATION.insert(mv);
                            }
                        };
                        return actionArgs;
                    }

-----

  然后作者给提了个修改建议，让加一个@Targetinstance，然后不对是否包含做区分，不过我一直没理解，他就自己改了，大体思路没变，就是在脚本中一定要有 @Targetinstance Throwable err这个参数，限制了必须有之后就不用在分开判断了，直接一起处理就好了，这样代码简洁了很多：

-----

                        addExtraTypeInfo(om.getTargetInstanceParameter(), THROWABLE_TYPE);
                        vr = validateArguments(om, actionArgTypes, Type.getArgumentTypes(getDescriptor()));
			

                            ArgumentProvider[] actionArgs = new ArgumentProvider[5];

                            actionArgs[0] = localVarArg(om.getTargetInstanceParameter(), THROWABLE_TYPE, throwableIndex);
                            actionArgs[1] = constArg(om.getClassNameParameter(), className.replace('/', '.'));
                            actionArgs[2] = constArg(om.getMethodParameter(), getName(om.isMethodFqn()));
                            actionArgs[3] = selfArg(om.getSelfParameter(), Type.getObjectType(className));
                            actionArgs[4] = new ArgumentProvider(asm, om.getDurationParameter()) {
                                @Override
                                public void doProvide() {
                                    MethodTrackingExpander.DURATION.insert(mv);
                                }
                            };

                            Label l = levelCheck(om, bcn.getClassName(true));

                            loadArguments(vr, actionArgTypes, isStatic(), actionArgs);

-----

  下面是脚本的写法：

-----
		@onmethod(clazz="/.*\.OnMethodTest/", method="args", location=@location(value=Kind.ERROR))
		public static void args(@self Object self, @return long retVal, @duration long dur, String a, long b, String[] c, int[] d, @Targetinstance Throwable err) {
		println("args");
		}
-----

  以下是最开始的查问题的时候，为了快，临时修改代码的记录：

-----

  子公司线上的一个项目偶尔会报一个异常，问题出在一个Filter里，由于Filter是由web容器调用的，所以全都没有走自己的代码，也不容易加日志，由于搬的位置比较近，我就帮着看看，先是想用BTrace拦截到Filter的参数：

```markdown

        @OnMethod(clazz = "com.....filter.CipherFilter", method = "doFilter", location = @Location(Kind.RETURN))
        public static void traceExecute(@ProbeClassName String name, @ProbeMethodName String method,
                                                                     AnyType servletRequest, AnyType servletResponse, AnyType chain){
            .......
        }

        @OnMethod(clazz = "com.....filter.CipherFilter", method = "doFilter", location = @Location(Kind.ERROR))
        public static void traceErrorExecuteWithoutParas(@ProbeClassName String name,@ProbeMethodName String method){
            ......
        }

        @OnMethod(clazz = "com.....filter.CipherFilter", method = "doFilter", location = @Location(Kind.ERROR))
        public static void traceErrorExecute(@ProbeClassName String name,@ProbeMethodName String method,
                                             AnyType servletRequest, AnyType servletResponse, AnyType chain){
            ......
        }

```

  然而发现，不报错时，traceExecute方法可以拿到参数。报错的时候，只有traceErrorExecuteWithoutParas会执行，traceErrorExecute完全没有输出，这就比较尴尬了，这项目之前都没有接触过，这第三方框架也没用过，如果连什么参数造成的错误都不知道，那就没什么可以参照的信息了。于是我就想看看，为什么参数没有输出，就fork了[btrace](https://github.com/saaavsaaa/btrace)的源码。
  在Instrumentor.java里找到了原因：
		
```markdown

            case ERROR:
                // <editor-fold defaultstate="collapsed" desc="Error Instrumentor">
                ErrorReturnInstrumentor eri = new ErrorReturnInstrumentor(cl, mv, mHelper, className, superName, access, name, desc) {
                    ValidationResult vr;
                    {
                        addExtraTypeInfo(om.getSelfParameter(), Type.getObjectType(className));
                        vr = validateArguments(om, actionArgTypes, new Type[]{THROWABLE_TYPE});
                    }

                    @Override
                    protected void onErrorReturn() {
                        ......

                            ArgumentProvider[] actionArgs = new ArgumentProvider[5];

                            actionArgs[0] = localVarArg(vr.getArgIdx(0), THROWABLE_TYPE, throwableIndex);
                            .......

                            loadArguments(actionArgs);

```

  源码中处理ERROR的情况时参数直接就忽略了，用了THROWABLE_TYPE（THROWABLE_TYPE = Throwable.class.getName()），或许BTrace提供了别的方法，或许我这种需求不多，总之，一时也没找到办法，就只好自己改了。Filter的doFilter方法没有返回值，我暂时就不处理返回值的情况，先解决问题，至于其他回头抽出空来再说了。
  其实如果不考虑很多情况，只是要拿到参数很容易：

```markdown

       vr = validateArguments(om, actionArgTypes, new Type[]{THROWABLE_TYPE});
       -->
       vr = validateArguments(om, actionArgTypes, new Type[]{THROWABLE_TYPE});


       ArgumentProvider[] actionArgs = new ArgumentProvider[5];
       ...
       -->
            if (vr.isAny()){
                callWithoutArgs(throwableIndex);
            } else {
                callWithArgs();
            }
            
           private void callWithArgs(){
                loadArguments(
                        vr, actionArgTypes, isStatic(),
                        constArg(om.getMethodParameter(), getName(om.isMethodFqn())),
                        constArg(om.getClassNameParameter(), className.replace("/", ".")),
        //                                    localVarArg(om.getReturnParameter(), probeRetType, retValIndex, boxReturnValue),
                        selfArg(om.getSelfParameter(), Type.getObjectType(className)),
                        new ArgumentProvider(asm, om.getDurationParameter()) {
                            @Override
                            public void doProvide() {
                                MethodTrackingExpander.DURATION.insert(mv);
                            }
                        }
                );
            }

            private void callWithoutArgs(int throwableIndex){
                ArgumentProvider[] actionArgs = new ArgumentProvider[5];

                actionArgs[0] = localVarArg(vr.getArgIdx(0), THROWABLE_TYPE, throwableIndex);
                actionArgs[1] = constArg(om.getClassNameParameter(), className.replace('/', '.'));
                actionArgs[2] = constArg(om.getMethodParameter(), getName(om.isMethodFqn()));
                actionArgs[3] = selfArg(om.getSelfParameter(), Type.getObjectType(className));
                actionArgs[4] = new ArgumentProvider(asm, om.getDurationParameter()) {
                    @Override
                    public void doProvide() {
                        MethodTrackingExpander.DURATION.insert(mv);
                    }
                };

                Label l = levelCheck(om, bcn.getClassName(true));

                loadArguments(actionArgs);
            }
       
```

  这样就可以拿到参数了：

-----

        ERROR class name ============com.....filter.CipherFilter
        ERROR class method ============doFilter
        ERROR time==1/3/18 2:35 PM
        /.../.../....action
        /.../.../....action

-----

  另外，从ServletRequest中拿到请求的url和参数之类的也稍有点啰嗦：
    
```markdown

        Object request = get(field("org.apache.catalina.connector.RequestFacade","request"), servletRequest);
        Object coyoteRequest = get(field("org.apache.catalina.connector.Request","coyoteRequest"), request);

        Object uri = get(field("org.apache.coyote.Request","uriMB"), coyoteRequest);
        println(get(field("org.apache.tomcat.util.buf.MessageBytes","strValue"), uri));
    
        Object path = get(field("org.apache.catalina.connector.Request","requestDispatcherPath"), request);
        println(get(field("org.apache.tomcat.util.buf.MessageBytes","strValue"), path));

```

微信公众号：

![Image](/ppp/20170902204445.jpg)
