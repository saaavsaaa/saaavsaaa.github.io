# 过滤器的路径匹配

最近在将项目中的一部分迁到围绕Spring Cloud构建的架构上，于是就想将鉴权等一些原本通过Servlet的Filter和Interceptor实现的此类功能迁移到用Zuul做的路由网关上，统一由网关处理。然而，这些过滤器并不都针对所有请求，之前是通过Servlet提供的配置方法进行配置的，但是ZuulFilter我并没找到类似的配置，又打算兼容原来的配置，就读了一下Spring的实现，打算直接拿来用，其实实现很简单，只是这两种方式匹配规则稍微有些不一样，这里只是做一下阅读笔记。

需要处理的是如下面的两种情况：

1.过滤器
```markdown
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
2.拦截器
```markdown
    public void addInterceptors(InterceptorRegistry registry) {
        LoginInterceptor appLoginInterceptor = new LoginInterceptor();
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/**/aaa/*");
        super.addInterceptors(registry);
    }
```
过滤器匹配请求的逻辑在ApplicationFilterFactory类中:
```markdown
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
```
这一段的逻辑就是组成一个匹配当前路径的过滤器链，取出所有在启动时初始化的过滤器的配置，用当前dispatcher和requestPath与之匹配，这里主要关注的是第二个判断，就是请求路径与过滤器配置路基的匹配，如果匹配就加入过滤器链，后续过滤器链执行时就会执行。matchFiltersURL方法：
```markdown   
        for (int i = 0; i < testPaths.length; i++) {
            if (matchFiltersURL(testPaths[i], requestPath)) {
                return true;
            }
        }
```
下面是其中的matchFiltersURL方法，上面两段代码的方法名差了一个s容易看漏，这方法虽然是真正进行判断的部分，但其实没啥好说的，几种情况注释的也很明白，先匹配精确路径，不成就匹配通配的路径，最后是通配的带扩展名的例如test.action：
```markdown
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
拦截器的匹配是从DispatcherServlet开始的，不过这一段是引子，不需要关注：
```markdown
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		for (HandlerMapping hm : this.handlerMappings) {
			if (logger.isTraceEnabled()) {
				logger.trace(
						"Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
			}
			HandlerExecutionChain handler = hm.getHandler(request);
			if (handler != null) {
				return handler;
			}
		}
		return null;
	}


	public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		Object handler = getHandlerInternal(request);

		...
	}
```
AbstractUrlHandlerMapping依然是引子，不过是比较接近的引子了：
```markdown
	@Override
	protected Object getHandlerInternal(HttpServletRequest request) throws Exception {
		String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
		Object handler = lookupHandler(lookupPath, request);
		...
	}


	protected Object lookupHandler(String urlPath, HttpServletRequest request) throws Exception {
		...

		// Pattern match?
		List<String> matchingPatterns = new ArrayList<String>();
		for (String registeredPattern : this.handlerMap.keySet()) {
			if (getPathMatcher().match(registeredPattern, urlPath)) {
				matchingPatterns.add(registeredPattern);
			}
			else if (useTrailingSlashMatch()) {
				if (!registeredPattern.endsWith("/") && getPathMatcher().match(registeredPattern + "/", urlPath)) {
					matchingPatterns.add(registeredPattern +"/");
				}
			}
		}
		...
	}

```
这次到真正的匹配逻辑了，getPathMatcher().match的实现就在AntPathMatcher的doMatch方法，实际上就是先匹配每一段路径的字符串，然后又重头来了一遍匹配每一段的正则：
```markdown
		if (path.startsWith(this.pathSeparator) != pattern.startsWith(this.pathSeparator)) {
			return false;
		}

		String[] pattDirs = tokenizePattern(pattern);
		if (fullMatch && this.caseSensitive && !isPotentialMatch(path, pattDirs)) {
			return false;
		}
```
isPotentialMatch这个方法名挺有意思，先粗略判断一下，是不是可能，可能了再仔细判断，这个思路不错，可以借鉴：
```markdown
	private boolean isPotentialMatch(String path, String[] pattDirs) {
		if (!this.trimTokens) {
			char[] pathChars = path.toCharArray();
			int pos = 0;
			for (String pattDir : pattDirs) {
				int skipped = skipSeparator(path, pos, this.pathSeparator);
				pos += skipped;
				skipped = skipSegment(pathChars, pos, pattDir);
				if (skipped < pattDir.length()) {
					if (skipped > 0) {
						return true;
					}
					return (pattDir.length() > 0) && isWildcardChar(pattDir.charAt(0));
				}
				pos += skipped;
			}
		}
		return true;
	}
```
```markdown
	private int skipSeparator(String path, int pos, String separator) {
		int skipped = 0;
		while (path.startsWith(separator, pos + skipped)) {
			skipped += separator.length();
		}
		return skipped;
	}
	private int skipSegment(char[] chars, int pos, String prefix) {
		int skipped = 0;
		for (char c : prefix.toCharArray()) {
			if (isWildcardChar(c)) {
				return skipped;
			}
			else if (pos + skipped >= chars.length) {
				return 0;
			}
			else if (chars[pos + skipped] == c) {
				skipped++;
			}
		}
		return skipped;
	}
```
可以发现，其实就是比较每一段拦截的请求路径的字符串了，isWildcardChar是判断配置的规则是不是通配符'*', '?', '{'的，skipSeparator里的循环应该是为了对多打了/的情况容错的吧，大概。接下来回到doMatch方法继续往下：
```markdown
		String[] pathDirs = tokenizePath(path);

		int pattIdxStart = 0;
		int pattIdxEnd = pattDirs.length - 1;
		int pathIdxStart = 0;
		int pathIdxEnd = pathDirs.length - 1;

		// Match all elements up to the first **
		while (pattIdxStart <= pattIdxEnd && pathIdxStart <= pathIdxEnd) {
			String pattDir = pattDirs[pattIdxStart];
			if ("**".equals(pattDir)) {
				break;
			}
			if (!matchStrings(pattDir, pathDirs[pathIdxStart], uriTemplateVariables)) {
				return false;
			}
			pattIdxStart++;
			pathIdxStart++;
		}
```
上面做了**和正则的匹配，matchStrings的代码就不贴了，也没什么好细看的。
```markdown
		if (pathIdxStart > pathIdxEnd) {
```
拦截到的路径已经匹配完了
```markdown
			// Path is exhausted, only match if rest of pattern is * or **'s
			if (pattIdxStart > pattIdxEnd) {
				return (pattern.endsWith(this.pathSeparator) ? path.endsWith(this.pathSeparator) :
						!path.endsWith(this.pathSeparator));
			}
```
配置的pattern已经匹配完时，请求的是否是以路径分隔符‘/’（一般情况下）结尾
```markdown
			if (!fullMatch) {
				return true;
			}
			if (pattIdxStart == pattIdxEnd && pattDirs[pattIdxStart].equals("*") && path.endsWith(this.pathSeparator)) {
				return true;
			}
```
配置的pattern刚巧匹配到最后，最后一段是*并且请求是以路径分隔符结尾的。如果上面上个判断都不是，就执行下面这个循环检查配置的且尚未用来匹配的部分是不是都是"**",如果不是，那么就判断配置的规则与当前请求不匹配：
```markdown
			for (int i = pattIdxStart; i <= pattIdxEnd; i++) {
				if (!pattDirs[i].equals("**")) {
					return false;
				}
			}
			return true;
		}
```
因为之前匹配过**，所以这里如果是配置的pattern先于请求路径用完，请求就是不匹配的。
```markdown
		else if (pattIdxStart > pattIdxEnd) {
			// String not exhausted, but pattern is. Failure.
			return false;
		}
		else if (!fullMatch && "**".equals(pattDirs[pattIdxStart])) {
			// Path start definitely matches due to "**" part in pattern.
			return true;
		}
```
pattern和path同时到最后了，要认真检查一下...
```markdown
		// up to last '**'
		while (pattIdxStart <= pattIdxEnd && pathIdxStart <= pathIdxEnd) {
			String pattDir = pattDirs[pattIdxEnd];
			if (pattDir.equals("**")) {
				break;
			}
			if (!matchStrings(pattDir, pathDirs[pathIdxEnd], uriTemplateVariables)) {
				return false;
			}
			pattIdxEnd--;
			pathIdxEnd--;
		}
		if (pathIdxStart > pathIdxEnd) {
			// String is exhausted
			for (int i = pattIdxStart; i <= pattIdxEnd; i++) {
				if (!pattDirs[i].equals("**")) {
					return false;
				}
			}
			return true;
		}
```
按说其实能走进下面这个循环的，基本上一定是前面判断**的时候break了循环过来的，需要判断**后面又配了什么，这代码判断的...和前面过滤器基本是没法合一用了，不过无所谓了。
```markdown
		while (pattIdxStart != pattIdxEnd && pathIdxStart <= pathIdxEnd) {
			int patIdxTmp = -1;
			for (int i = pattIdxStart + 1; i <= pattIdxEnd; i++) {
				if (pattDirs[i].equals("**")) {
					patIdxTmp = i;
					break;
				}
			}
			if (patIdxTmp == pattIdxStart + 1) {
				// '**/**' situation, so skip one
				pattIdxStart++;
				continue;
			}
			// Find the pattern between padIdxStart & padIdxTmp in str between
			// strIdxStart & strIdxEnd
			int patLength = (patIdxTmp - pattIdxStart - 1);
			int strLength = (pathIdxEnd - pathIdxStart + 1);
			int foundIdx = -1;

			strLoop:
			for (int i = 0; i <= strLength - patLength; i++) {
				for (int j = 0; j < patLength; j++) {
					String subPat = pattDirs[pattIdxStart + j + 1];
					String subStr = pathDirs[pathIdxStart + i + j];
					if (!matchStrings(subPat, subStr, uriTemplateVariables)) {
						continue strLoop;
					}
				}
				foundIdx = pathIdxStart + i;
				break;
			}

			if (foundIdx == -1) {
				return false;
			}

			pattIdxStart = patIdxTmp;
			pathIdxStart = foundIdx + patLength;
		}

		for (int i = pattIdxStart; i <= pattIdxEnd; i++) {
			if (!pattDirs[i].equals("**")) {
				return false;
			}
		}

		return true;
	}
```

微信公众号：

				![Image](/ppp/20170902204445.jpg)


[方便修改用的传送门](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/FilterRegistrationBean-And-InterceptorRegistry-Check-Path.md)
