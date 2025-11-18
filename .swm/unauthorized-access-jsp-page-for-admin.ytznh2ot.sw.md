---
title: Unauthorized Access JSP Page for Admin
---
# Introduction

This document explains the design and implementation of the unauthorized access JSP page for the admin section in Shopizer. We will cover:

1. Why the <SwmPath>[shopizer/…/admin/unauthorized.jsp](shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp)</SwmPath> page exists and what it displays.
2. How the page prevents caching to avoid stale content.
3. The use of tag libraries and their role in the page.
4. The styling approach and resource linking.
5. The message shown to the user and its context.

# purpose of the <SwmPath>[shopizer/…/admin/unauthorized.jsp](shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp)</SwmPath> page

The <SwmPath>[shopizer/…/admin/unauthorized.jsp](shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp)</SwmPath> page is designed to inform users when they attempt an unauthorized action in the admin area, specifically when dual login on the same browser is detected and not allowed. This is a security measure to prevent session conflicts or unauthorized access.

# preventing caching of the page

The page sets HTTP headers to disable caching:

- Character encoding is set to <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" pos="6:6:8" line-data="		response.setCharacterEncoding(&quot;UTF-8&quot;);">`UTF-8`</SwmToken>.
- <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" pos="7:6:8" line-data="		response.setHeader(&quot;Cache-Control&quot;, &quot;no-cache&quot;);">`Cache-Control`</SwmToken> and Pragma headers are set to <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" pos="7:13:15" line-data="		response.setHeader(&quot;Cache-Control&quot;, &quot;no-cache&quot;);">`no-cache`</SwmToken>.
- Expires header is set to -1.

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="1">

---

This ensures that browsers do not cache the unauthorized page, forcing them to fetch a fresh version each time. This is important because authorization status can change, and showing a cached unauthorized page could confuse users or block legitimate access.

```java server pages
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

<html>

	<%
		response.setCharacterEncoding("UTF-8");
		response.setHeader("Cache-Control", "no-cache");
		response.setHeader("Pragma", "no-cache");
		response.setDateHeader("Expires", -1);
	%>
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="25">

---

Additionally, meta tags in the HTML head reinforce <SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" pos="26:15:17" line-data="		&lt;meta http-equiv=&quot;Pragma&quot; content=&quot;no-cache&quot;&gt;">`no-cache`</SwmToken> behavior at the browser level.

```java server pages
	<head>
		<meta http-equiv="Pragma" content="no-cache">
			<meta http-equiv="expires" content="0">
				<title><s:message code="label.storeadministration"
						text="Store administration" />
				</title>
```

---

</SwmSnippet>

# tag libraries usage

The page imports several JSP tag libraries:

- JSTL core and functions for standard JSP operations.
- Spring tags for message resolution and form handling.
- Spring Security tags for security-related JSP features.

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="12">

---

These libraries enable dynamic content rendering, internationalization support, and integration with Spring Security for authorization checks.

```java server pages
	<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
	<%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn"%>
	<%@ taglib uri="http://www.springframework.org/tags" prefix="s"%>
	<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
	<%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec" %>

	<%@page contentType="text/html"%>
	<%@page pageEncoding="UTF-8"%>
```

---

</SwmSnippet>

# styling and resource linking

Styling is done inline and via linked CSS files:

- Inline CSS defines layout and colors for elements like the login container and labels.
- External CSS files include Bootstrap and Shopizer-specific stylesheets, linked using JSTL's `<c:`<SwmToken path="shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" pos="56:7:7" line-data="					href=&quot;&lt;c:url value=&quot;/resources/css/bootstrap/css/sm-bootstrap.css&quot; /&gt;&quot;">`url`</SwmToken>`>` to ensure correct URL resolution.

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="35">

---

This combination provides a consistent look and feel aligned with the rest of the admin UI.

```java server pages
<style type=text/css>
#logon {
	margin: 0px auto;
	width: 550px
}


#controls {
	margin-left: -50px;
	margin-top: 30px;
}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="50">

---

&nbsp;

```java server pages
</style>




				<link
					href="<c:url value="/resources/css/bootstrap/css/sm-bootstrap.css" />"
					rel="stylesheet" />
				<link
					href="<c:url value="/resources/css/sm-bootstrap-responsive.css" />"
					rel="stylesheet" />
				<link href="<c:url value="/resources/css/shopizer.css" />"
					rel="stylesheet" />
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="65">

---

&nbsp;

```java server pages
				<style type=text/css>
.sm label {
	color: #EBEBEB;
	font-size: 16px;
}

.sm a {
	color: #EBEBEB;
	font-size: 16px;
}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="80">

---

&nbsp;

```java server pages
</style>


	</head>

	<body>

		<div id="tabbable" class="sm">
```

---

</SwmSnippet>

# displayed message and structure

The page structure is simple:

- A container div centers the content.
- Inside, a heading displays a message indicating that dual login on the same browser is not authorized.
- The message uses Spring's message tag to allow localization and fallback text.

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp" line="89">

---

This direct message informs the user why access is denied without extra fluff.

```java server pages
			<br />
			<br />

			<div id=logon>

				<div class="row">
					<h3>
					<s:message code="message.login.duallogin" text="Dual login not authorized on the same browser"/>
					
					</h3>
				</div>

			</div>
		</div>
	</body>
</html>
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
