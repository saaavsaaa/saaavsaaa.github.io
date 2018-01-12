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