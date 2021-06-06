# SpringMVC

当服务器启动后，读取web.xml，此文件中包含了Servlet和Filter，如果配置load-on-startup，启动时就会实例化

## DispatcherServlet 初始化

![](D:\0_LeargingSummary\SpringMvc\images\QQ截图20200516092700.png)

DispatcherServlet本身就是继承自HttpServlet,tomcat启动的时候，会调用init方法

```java
public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
    
    public final void init() throws ServletException {

		// Set bean properties from init parameters.
        //设置初始化参数
		PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
		if (!pvs.isEmpty()) {
			try {
                //将dispatcherServlet进行包装
				BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
				ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
				bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
                //初始化dispatcherServlet
				initBeanWrapper(bw);
                //将初始化值设置到dispatcherServlet中
				bw.setPropertyValues(pvs, true);
			}
			catch (BeansException ex) {
				if (logger.isErrorEnabled()) {
					logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
				}
				throw ex;
			}
		}

		// Let subclasses do whatever initialization they like.
        //模版方法，调用子类的实现,执行完后等待请求
		initServletBean();
	}
}
```

```java
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
    
protected final void initServletBean() throws ServletException {
    //打印日志
		getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
		if (logger.isInfoEnabled()) {
			logger.info("Initializing Servlet '" + getServletName() + "'");
		}
    //开始时间
		long startTime = System.currentTimeMillis();

		try {
            //这里面初始化webapplicationcontext
			this.webApplicationContext = initWebApplicationContext();
            //模版方法
			initFrameworkServlet();
		}
		catch (ServletException | RuntimeException ex) {
			logger.error("Context initialization failed", ex);
			throw ex;
		}

		if (logger.isDebugEnabled()) {
			String value = this.enableLoggingRequestDetails ?
					"shown which may lead to unsafe logging of potentially sensitive data" :
					"masked to prevent unsafe logging of potentially sensitive data";
			logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
					"': request parameters and headers will be " + value);
		}

		if (logger.isInfoEnabled()) {
			logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
		}
	}
}
```

在初始化webapplicationcontext中发布context刷新事件，对应的监听器调用dispatcherServlet的onRefresh方法

```java
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}

	protected void initStrategies(ApplicationContext context) {
        //初始化文件解析器
		initMultipartResolver(context);
        //初始化国际化解析器
		initLocaleResolver(context);
        //主题
		initThemeResolver(context);
        //处理器映射，*****这里选择最常见的解析器进入**********
		initHandlerMappings(context);
        //处理器适配
		initHandlerAdapters(context);
        //异常解析器
		initHandlerExceptionResolvers(context);
       // RequestToViewName解析器
		initRequestToViewNameTranslator(context);
        //视图解析器
		initViewResolvers(context);
        //FlashMap管理器
		initFlashMapManager(context);
	}
```

进入到initHandlerMappings(context);

```java
private void initHandlerMappings(ApplicationContext context) {
		this.handlerMappings = null;
		//检测所有的handlerMappings
		if (this.detectAllHandlerMappings) {
			// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
			Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.handlerMappings = new ArrayList<>(matchingBeans.values());
				// We keep HandlerMappings in sorted order.
				AnnotationAwareOrderComparator.sort(this.handlerMappings);
			}
		}
		else {
			try {
				HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
				this.handlerMappings = Collections.singletonList(hm);
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Ignore, we'll add a default HandlerMapping later.
			}
		}

    //如果未找到其他映射，请通过注册默认的HandlerMapping来确保至少有一个HandlerMapping。
		if (this.handlerMappings == null) {
            //进入此行代码
       //RequestMappingHandlerMapping,BeanNameUrlHandlerMapping,RouterFunctionMapping
			this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
			if (logger.isTraceEnabled()) {
				logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
						"': using default strategies from DispatcherServlet.properties");
			}
		}
	}
```

```java
//默认获得策略	
protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {
		String key = strategyInterface.getName();
		String value = defaultStrategies.getProperty(key);
		if (value != null) {
			String[] classNames = StringUtils.commaDelimitedListToStringArray(value);
			List<T> strategies = new ArrayList<>(classNames.length);
            //循环遍历创建HandlerMapping
			for (String className : classNames) {
				try {
					Class<?> clazz = ClassUtils.forName(className, DispatcherServlet.class.getClassLoader());
					Object strategy = createDefaultStrategy(context, clazz);
					strategies.add((T) strategy);
				}
				catch (ClassNotFoundException ex) {
					throw new BeanInitializationException(
							"Could not find DispatcherServlet's default strategy class [" + className +
							"] for interface [" + key + "]", ex);
				}
				catch (LinkageError err) {
					throw new BeanInitializationException(
							"Unresolvable class definition for DispatcherServlet's default strategy class [" +
							className + "] for interface [" + key + "]", err);
				}
			}
			return strategies;
		}
		else {
			return new LinkedList<>();
		}
	}
```

## DispatcherServlet处理请求

```java
//FrameworkServlet
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		//获得启动时间
		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;
		//获得之前的本地语言上下文
		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    	//根据当前request创建一个本地上下文SimpleLocaleContext
		LocaleContext localeContext = buildLocaleContext(request);
		//获得之前的request属性
		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    //如果当前的preivousAttributes为空，就创建ServletRequestAttributes，否则就返回null
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);
//获得异步管理器
		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());
//初始化本地上下文，设置request,requestAttributes
		initContextHolders(request, localeContext, requestAttributes);

		try {
            //处理请求
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}
```

> ### FlashMap作用：FlashMap提供了一种方法，用于一个请求存储打算在另一个请求中使用的属性。 从一个网址重定向到另一个网址时，这是最常见的需求-例如 发布/重定向/获取模式。 FlashMap在重定向之前（通常在会话中）保存，并在重定向后可用，并立即删除。

```java
//DispatcherServlet
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			attributesSnapshot = new HashMap<>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// Make framework objects available to handlers and view objects.
    //使框架对象可以处理程序并查看对象
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		if (this.flashMapManager != null) {
            //如果有Http Session，则检索保存flashMap
			FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
			if (inputFlashMap != null) {
				request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
			}
			request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
			request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
		}

		try {
            //核心方法，用于分发请求，进行处理
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
		}
	}
```

```java
//DispatcherServlet的核心处理方法
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   //定义一个处理请求
   HttpServletRequest processedRequest = request;
    //定义一个执行链
   HandlerExecutionChain mappedHandler = null;
    //用于判断是否是文件上传请求
   boolean multipartRequestParsed = false;
	//获得异步管理器
   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

   try {
       //定义模型视图
      ModelAndView mv = null;
       //用于保存发生的异常
      Exception dispatchException = null;

      try {
         //检查是否有上传需求
         processedRequest = checkMultipart(request);
         multipartRequestParsed = (processedRequest != request);

         // Determine handler for the current request.
         //获得当前请求处理器链
         mappedHandler = getHandler(processedRequest);
          //如果没有对应的处理器
         if (mappedHandler == null) {
             //返回404或是抛异常
            noHandlerFound(processedRequest, response);
            return;
         }

          //获得当前请求的处理器适配器
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

         // Process last-modified header, if supported by the handler.
         //处理last-modified请求头，用于判断请求内容是否发生修改
         String method = request.getMethod();
         boolean isGet = "GET".equals(method);
          
          //这里只处理get请求或是head请求
         if (isGet || "HEAD".equals(method)) {
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
               return;
            }
         }
         //应用拦截器的前置方法，如果有任何一个不过，就会返回。内部就是用的for循环
         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
         }

         // Actually invoke the handler.
          //真正执行处理器，此方法的返回值是ModelAndView对象，封装了模型和视图
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

         if (asyncManager.isConcurrentHandlingStarted()) {
            return;
         }
          
		//应用默认的视图
         applyDefaultViewName(processedRequest, mv);
		//拦截器的后置方法
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
          // 保存异常信息
         dispatchException = ex;
      }
      catch (Throwable err) {
         // As of 4.3, we're processing Errors thrown from handler methods as well,
         // making them available for @ExceptionHandler methods and other scenarios.
         dispatchException = new NestedServletException("Handler dispatch failed", err);
      }
      //处理结果，可以是视图处理或者异常处理
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }
   catch (Exception ex) {
      // 链式执行拦截器链的afterCompletion方法
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
   }
   catch (Throwable err) {
       
      triggerAfterCompletion(processedRequest, response, mappedHandler,
            new NestedServletException("Handler processing failed", err));
   }
   finally {
      if (asyncManager.isConcurrentHandlingStarted()) {
         // Instead of postHandle and afterCompletion
         if (mappedHandler != null) {
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
         }
      }
      else {
         // Clean up any resources used by a multipart request.
         if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
         }
      }
   }
}
```

> ### HTTP响应头部中的Last-modified字段表示请求的资源的最后被修改的时间，这个字段的作copy用就是用于缓存服务器机制，用于判断缓存服务器中的资源是否过期，是否需要从源服务器更新。

以上是doDispatch大致流程,具体看流程图

## 处理器适配器的详细处理过程

```java
protected ModelAndView handleInternal(HttpServletRequest request,
      HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

   ModelAndView mav;
   //检查给定请求获得受支持的方法和所需的会话（如果需要）
   checkRequest(request);

    //如果需要在同步代码块中执行invokeHandlerMethod
   if (this.synchronizeOnSession) {
      HttpSession session = request.getSession(false);
      if (session != null) {
         Object mutex = WebUtils.getSessionMutex(session);
         synchronized (mutex) {
            mav = invokeHandlerMethod(request, response, handlerMethod);
         }
      }
      else {
         // No HttpSession available -> no mutex necessary
         mav = invokeHandlerMethod(request, response, handlerMethod);
      }
   }
   else {
      //不需要同步会话
      mav = invokeHandlerMethod(request, response, handlerMethod);
   }

   if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
      if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
         applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
      }
      else {
         prepareResponse(response);
      }
   }

   return mav;
}
```

### invokeHandlerMethod执行处理器方法

```java
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
      HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
	//将request和response封装成webrequest
   ServletWebRequest webRequest = new ServletWebRequest(request, response);
   try {
       //对注解@InitBinder的处理，该工厂用于获取处理器方法对应的WebDataBinder组件
      WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
      //返回一个模型工厂， 获取当前处理器方法对应的Model工厂，该工厂用于获取处理器方法对应的model
      ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

       //创建一个ServletInvocableHandlerMethod对象
      ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
      if (this.argumentResolvers != null) {
          //设置参数解析器
         invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
      }
      if (this.returnValueHandlers != null) {
 //设置返回值处理器
          invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
      }
      //设置DataBinder工厂
      invocableMethod.setDataBinderFactory(binderFactory);
      // 设置参数名获取器，用于获取方法上的参数名
      invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);
	//ModelAndView容器
      ModelAndViewContainer mavContainer = new ModelAndViewContainer();
      mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
      modelFactory.initModel(webRequest, mavContainer, invocableMethod);
      mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);
	//准备异步的一些相关操作
      AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
      //有效时间
      asyncWebRequest.setTimeout(this.asyncRequestTimeout);

      WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
      //设置线程池
      asyncManager.setTaskExecutor(this.taskExecutor);
      asyncManager.setAsyncWebRequest(asyncWebRequest);
      asyncManager.registerCallableInterceptors(this.callableInterceptors);
      asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

      if (asyncManager.hasConcurrentResult()) {
         Object result = asyncManager.getConcurrentResult();
         mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
         asyncManager.clearConcurrentResult();
         LogFormatUtils.traceDebug(logger, traceOn -> {
            String formatted = LogFormatUtils.formatValue(result, !traceOn);
            return "Resume with async result [" + formatted + "]";
         });
         invocableMethod = invocableMethod.wrapConcurrentResult(result);
      }
//调用处理器方法并处理返回值，如果有返回视图，就将视图名保存到上面的mavContainer中
      invocableMethod.invokeAndHandle(webRequest, mavContainer);
      if (asyncManager.isConcurrentHandlingStarted()) {
         return null;
      }
		//封装获得ModelAndView
      return getModelAndView(mavContainer, modelFactory, webRequest);
   }
   finally {
      webRequest.requestCompleted();
   }
}
```