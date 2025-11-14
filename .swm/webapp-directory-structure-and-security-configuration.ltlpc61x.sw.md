---
title: Webapp Directory Structure and Security Configuration
---
# Overview of the webapp directory

The webapp directory in the Shop Application contains all the web-facing resources and configuration files essential for the user interface and web functionality. It includes JSP files, tag libraries, JavaScript, CSS, and the <SwmPath>[shopizer/…/WEB-INF/web.xml](shopizer/sm-shop/src/main/webapp/WEB-INF/web.xml)</SwmPath> deployment descriptor, which collectively define the structure, appearance, and behavior of the web pages.

This directory organizes common UI components and administrative interface elements that support the shopping experience for customers as well as management tasks for administrators. It is a central location where developers can work on the visual and interactive parts of the application.

# Purpose and role in deployment

The webapp folder is critical for packaging the application as a WAR file, which is then deployed on a servlet container such as Apache Tomcat. Without this directory, the application would lack the necessary web resources and configuration to function properly in a web environment.

# How developers use the webapp directory

Developers modify and extend the webapp directory to update the user interface and web interactions. Changes here directly affect key features such as the shopping cart, catalogue browsing, checkout process, and administrative management. Additionally, configuration files within this directory, including security settings, control access and authentication mechanisms.

# Example: Checkout page and security configuration

For instance, the <SwmPath>[shopizer/…/checkout/checkout.jsp](shopizer/sm-shop/src/main/webapp/pages/shop/common/checkout/checkout.jsp)</SwmPath> file located in the webapp directory implements the checkout page's user interface, enabling customers to complete their purchases. Security configurations like <SwmPath>[shopizer/…/appServlet/shopizer-security.xml](shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml)</SwmPath> define URL patterns, access rules, and service implementations that manage customer authentication and authorization for both the shop and admin sections.

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" line="49">

---

The webapp directory includes security configuration files that specify how web requests are routed and secured. For example, admin endpoints are defined under the URL pattern <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="49:7:9" line-data="	&lt;http pattern=&quot;/admin/**&quot; auto-config=&quot;true&quot; use-expressions=&quot;true&quot; authentication-manager-ref=&quot;userAuthenticationManager&quot;&gt;">`/admin/**`</SwmToken> with specific pages accessible to all users, such as <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="50:9:14" line-data="		&lt;intercept-url pattern=&quot;/admin/logon.html*&quot; access=&quot;permitAll&quot; /&gt;">`/admin/logon.html`</SwmToken> and <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="51:9:14" line-data="		&lt;intercept-url pattern=&quot;/admin/denied.html&quot; access=&quot;permitAll&quot;/&gt;">`/admin/denied.html`</SwmToken>. Other admin pages require users to have the <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="55:19:19" line-data="		&lt;intercept-url pattern=&quot;/admin&quot; access=&quot;hasRole(&#39;AUTH&#39;)&quot; /&gt;">`AUTH`</SwmToken> role. Login and logout URLs, as well as access denied handlers, are also configured here.

```xml
	<http pattern="/admin/**" auto-config="true" use-expressions="true" authentication-manager-ref="userAuthenticationManager">
		<intercept-url pattern="/admin/logon.html*" access="permitAll" />
		<intercept-url pattern="/admin/denied.html" access="permitAll"/>
		<intercept-url pattern="/admin/unauthorized.html" access="permitAll"/>
		<intercept-url pattern="/admin/users/resetPassword.html*" access="permitAll" />
		<intercept-url pattern="/admin/users/resetPasswordSecurityQtn.html*" access="permitAll" /> 
		<intercept-url pattern="/admin" access="hasRole('AUTH')" />
		<intercept-url pattern="/admin/" access="hasRole('AUTH')" />
		<intercept-url pattern="/admin/*.html*" access="hasRole('AUTH')" />
		<intercept-url pattern="/admin/*/*.html*" access="hasRole('AUTH')" />
		<intercept-url pattern="/admin/*/*/*.html*" access="hasRole('AUTH')" />

		
		<form-login 
			login-processing-url="/admin/j_spring_security_check" 
			login-page="/admin/logon.html"
			authentication-success-handler-ref="userAuthenticationSuccessHandler"
			authentication-failure-url="/admin/logon.html?login_error=true"
			default-target-url="/admin/home.html" />
			
			
		<logout invalidate-session="true" 
			logout-success-url="/admin/home.html" 
			logout-url="/admin/j_spring_security_logout" />
		<access-denied-handler ref="adminAccessDenied"/>
	</http>
```

---

</SwmSnippet>

This snippet from <SwmPath>[shopizer/…/appServlet/shopizer-security.xml](shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml)</SwmPath> shows the admin security configuration, including login processing, logout handling, and access control rules.

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" line="78">

---

Similarly, customer-facing endpoints are defined under the URL pattern <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="78:7:9" line-data="	&lt;http pattern=&quot;/shop/**&quot; auto-config=&quot;true&quot; use-expressions=&quot;true&quot; authentication-manager-ref=&quot;customerAuthenticationManager&quot;&gt;">`/shop/**`</SwmToken>. Public pages like <SwmPath>[shopizer/…/entity/shop/](shopizer/sm-shop/src/main/java/com/salesmanager/web/entity/shop/)</SwmPath>, <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="83:9:16" line-data="		&lt;intercept-url pattern=&quot;/shop/customer/logon.html*&quot; access=&quot;permitAll&quot; /&gt;">`/shop/customer/logon.html`</SwmToken>, and <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="84:9:16" line-data="		&lt;intercept-url pattern=&quot;/shop/customer/registration.html*&quot; access=&quot;permitAll&quot; /&gt;">`/shop/customer/registration.html`</SwmToken> are accessible to all users, while pages under <SwmPath>[shopizer/…/business/customer/](shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/customer/)</SwmPath>`**` require the <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml" pos="88:21:21" line-data="		&lt;intercept-url pattern=&quot;/shop/customer&quot; access=&quot;hasRole(&#39;AUTH_CUSTOMER&#39;)&quot; /&gt;">`AUTH_CUSTOMER`</SwmToken> role. Logout URLs and access denied redirects are also specified to manage customer sessions and security.

```xml
	<http pattern="/shop/**" auto-config="true" use-expressions="true" authentication-manager-ref="customerAuthenticationManager">

		<intercept-url pattern="/shop" access="permitAll" />
		<intercept-url pattern="/shop/" access="permitAll" />
		<intercept-url pattern="/shop/**" access="permitAll" />
		<intercept-url pattern="/shop/customer/logon.html*" access="permitAll" />
		<intercept-url pattern="/shop/customer/registration.html*" access="permitAll" />
		<intercept-url pattern="/shop/customer/customLogon.html*" access="permitAll" />
		<intercept-url pattern="/shop/customer/denied.html" access="permitAll"/>
		<intercept-url pattern="/shop/customer/j_spring_security_check" access="permitAll"/>
		<intercept-url pattern="/shop/customer" access="hasRole('AUTH_CUSTOMER')" />
		<intercept-url pattern="/shop/customer/" access="hasRole('AUTH_CUSTOMER')" />
		<intercept-url pattern="/shop/customer/*.html*" access="hasRole('AUTH_CUSTOMER')" />
		<intercept-url pattern="/shop/customer/*.html*" access="hasRole('AUTH_CUSTOMER')" />
		<intercept-url pattern="/shop/customer/*/*.html*" access="hasRole('AUTH_CUSTOMER')" />
		<intercept-url pattern="/shop/customer/*/*/*.html*" access="hasRole('AUTH_CUSTOMER')" />

			
		<logout invalidate-session="false" 
			logout-success-url="/shop/" 
			logout-url="/shop/customer/j_spring_security_logout" />
		<access-denied-handler error-page="/shop/"/>
	</http>
	
```

---

</SwmSnippet>

This snippet from <SwmPath>[shopizer/…/appServlet/shopizer-security.xml](shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml)</SwmPath> details the security configuration for customer endpoints, including access rules and logout behavior.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
