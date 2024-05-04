# #DelegatingFilterProxy

```java
package org.springframework.web.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;

public class DelegatingFilterProxy extends GenericFilterBean {

	@Nullable
	private String contextAttribute;

	@Nullable
	private WebApplicationContext webApplicationContext;

	@Nullable
	private String targetBeanName;

	private boolean targetFilterLifecycle = false;

	@Nullable
	private volatile Filter delegate;

	private final Object delegateMonitor = new Object();

	public DelegatingFilterProxy() {
	}


	@Override
	protected void initFilterBean() throws ServletException {
		synchronized (this.delegateMonitor) {
			if (this.delegate == null) {
				// If no target bean name specified, use filter name.
				if (this.targetBeanName == null) {
					this.targetBeanName = getFilterName();
				}
				// Fetch Spring root application context and initialize the delegate early,
				// if possible. If the root application context will be started after this
				// filter proxy, we'll have to resort to lazy initialization.
				WebApplicationContext wac = findWebApplicationContext();
				if (wac != null) {
					this.delegate = initDelegate(wac);
				}
			}
		}
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		// Lazily initialize the delegate if necessary.
		Filter delegateToUse = this.delegate;
		if (delegateToUse == null) {
			synchronized (this.delegateMonitor) {
				delegateToUse = this.delegate;
				if (delegateToUse == null) {
					WebApplicationContext wac = findWebApplicationContext();
					if (wac == null) {
						throw new IllegalStateException("No WebApplicationContext found: " +
								"no ContextLoaderListener or DispatcherServlet registered?");
					}
					delegateToUse = initDelegate(wac);
				}
				this.delegate = delegateToUse;
			}
		}

		// Let the delegate perform the actual doFilter operation.
		invokeDelegate(delegateToUse, request, response, filterChain);
	}

	@Override
	public void destroy() {
		Filter delegateToUse = this.delegate;
		if (delegateToUse != null) {
			destroyDelegate(delegateToUse);
		}
	}

	@Nullable
	protected WebApplicationContext findWebApplicationContext() {
		if (this.webApplicationContext != null) {
			// The user has injected a context at construction time -> use it...
			if (this.webApplicationContext instanceof ConfigurableApplicationContext) {
				ConfigurableApplicationContext cac = (ConfigurableApplicationContext) this.webApplicationContext;
				if (!cac.isActive()) {
					// The context has not yet been refreshed -> do so before returning it...
					cac.refresh();
				}
			}
			return this.webApplicationContext;
		}
		String attrName = getContextAttribute();
		if (attrName != null) {
			return WebApplicationContextUtils.getWebApplicationContext(getServletContext(), attrName);
		}
		else {
			return WebApplicationContextUtils.findWebApplicationContext(getServletContext());
		}
	}

	protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
		String targetBeanName = getTargetBeanName();
		Assert.state(targetBeanName != null, "No target bean name set");
		Filter delegate = wac.getBean(targetBeanName, Filter.class);
		if (isTargetFilterLifecycle()) {
			delegate.init(getFilterConfig());
		}
		return delegate;
	}

	protected void invokeDelegate(
			Filter delegate, ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		delegate.doFilter(request, response, filterChain);
	}

	protected void destroyDelegate(Filter delegate) {
		if (isTargetFilterLifecycle()) {
			delegate.destroy();
		}
	}

}
```
