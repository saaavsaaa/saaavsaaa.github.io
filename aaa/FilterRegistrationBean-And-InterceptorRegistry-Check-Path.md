# 过滤器的路径匹配

最近在将项目中的一部分迁到围绕Spring Cloud构建的架构上，于是就想将鉴权等一些原本通过Servlet的Filter和Interceptor实现的此类功能迁移到用Zuul做的路由网关上，统一由网关处理。然而，这些过滤器并不都针对所有请求，之前是通过Servlet提供的配置方法进行配置的，但是ZuulFilter我并没找到类似的配置，又打算兼容原来的配置，就读了一下Spring的实现，其实实现很简单，这里只是做一下阅读笔记。

需要处理的主要是想下面的两种：
```markdown
1.过滤器
@Configuration
@PropertySource("classpath:filter.properties")
public class FilterRegister {
    
    @Value("${filter.aaa}")
    private String path;
    
   @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(new AAAFilter());
        registrationBean.addUrlPatterns(path);
        return registrationBean;
    }
}
```
```markdown
2.拦截器
    public void addInterceptors(InterceptorRegistry registry) {
        LoginInterceptor appLoginInterceptor = new LoginInterceptor();
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/**/aaa/*");
        super.addInterceptors(registry);
    }
```


```markdown
过滤器匹配请求的逻辑在：
ApplicationFilterFactory

    public static ApplicationFilterChain createFilterChain(ServletRequest request,
            Wrapper wrapper, Servlet servlet) {

	......

        for (int i = 0; i < filterMaps.length; i++) {
            if (!matchDispatcher(filterMaps[i] ,dispatcher)) {
                continue;
            }
            if (!matchFiltersURL(filterMaps[i], requestPath))
                continue;
            ApplicationFilterConfig filterConfig = (ApplicationFilterConfig)
                context.findFilterConfig(filterMaps[i].getFilterName());
            if (filterConfig == null) {
                // FIXME - log configuration problem
                continue;
            }
            filterChain.addFilter(filterConfig);
        }

	......
    }
    
    matchFiltersURL方法：
        for (int i = 0; i < testPaths.length; i++) {
            if (matchFiltersURL(testPaths[i], requestPath)) {
                return true;
            }
        }
    matchFiltersURL方法：
    
    private static boolean matchFiltersURL(String testPath, String requestPath) {

        if (testPath == null)
            return false;

        // Case 1 - Exact Match
        if (testPath.equals(requestPath))
            return true;

        // Case 2 - Path Match ("/.../*")
        if (testPath.equals("/*"))
            return true;
        if (testPath.endsWith("/*")) {
            if (testPath.regionMatches(0, requestPath, 0,
                                       testPath.length() - 2)) {
                if (requestPath.length() == (testPath.length() - 2)) {
                    return true;
                } else if ('/' == requestPath.charAt(testPath.length() - 2)) {
                    return true;
                }
            }
            return false;
        }

        // Case 3 - Extension Match
        if (testPath.startsWith("*.")) {
            int slash = requestPath.lastIndexOf('/');
            int period = requestPath.lastIndexOf('.');
            if ((slash >= 0) && (period > slash)
                && (period != requestPath.length() - 1)
                && ((requestPath.length() - period)
                    == (testPath.length() - 1))) {
                return (testPath.regionMatches(2, requestPath, period + 1,
                                               testPath.length() - 2));
            }
        }

        // Case 4 - "Default" Match
        return false; // NOTE - Not relevant for selecting filters

    }
    
```

```markdown
第二种
```

```markdown
[**默认** *AAA* ***AAA***](https://github.com/saaavsaaa/saaavsaaa.github.io/aaa/aaa.md)

1.aaa
2.bbb

'aaa'
```

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/FilterRegistrationBean-And-InterceptorRegistry-Check-Path.md)
