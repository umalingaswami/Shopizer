---
title: Webapp Core Model Overview
---
# Overview of Webapp Core Model

The Webapp directory within the Core Model serves as the central repository for all web application resources that enable the user interface and interaction in the platform. It organizes essential components such as configuration files, JSP pages, tag libraries, and static assets like JavaScript and CSS, which collectively support the presentation layer of both the storefront and administrative interfaces.

# Configuration Files

Key configuration files such as <SwmPath>[shopizer/…/WEB-INF/web.xml](shopizer/sm-shop/src/main/webapp/WEB-INF/web.xml)</SwmPath> define how the web application is deployed and behaves within the servlet container. These files specify servlet mappings, context parameters, and other deployment descriptors that control the lifecycle and routing of web requests. Additionally, security configurations like <SwmPath>[shopizer/…/appServlet/shopizer-security.xml](shopizer/sm-shop/src/main/webapp/WEB-INF/spring/appServlet/shopizer-security.xml)</SwmPath> are located here, managing authentication and authorization rules for different user roles.

# JSP Pages and Tag Libraries

JSP files in the Webapp directory are responsible for rendering dynamic content to users. They leverage tag libraries to provide reusable UI components, which promote modularity and consistency across the application. These JSPs cover various functionalities, including product display, category browsing, checkout processes, and administrative tasks, ensuring a cohesive user experience.

# Static Assets

Static resources such as JavaScript and CSS files reside in the Webapp directory to enhance the user interface and interactivity. For example, JavaScript libraries like <SwmPath>[shopizer/…/js/bootstrap.js](shopizer/sm-shop/src/main/webapp/resources/templates/bootstrap3/js/bootstrap.js)</SwmPath> enable UI elements such as modals, dropdowns, and carousels, contributing to a responsive and engaging storefront and admin interface.

# Webapp Endpoints and Security

The Webapp Core Model defines distinct endpoint patterns secured via Spring Security configurations. Administrative endpoints are mapped under <SwmPath>[shopizer/…/web/admin/](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/)</SwmPath>`**` and require authenticated users with the 'AUTH' role. Public access is granted to specific pages like login and error handling JSPs. Customer-facing shop endpoints fall under <SwmPath>[shopizer/…/entity/shop/](shopizer/sm-shop/src/main/java/com/salesmanager/web/entity/shop/)</SwmPath>`**`, with public access to general pages and restricted access to authenticated customers under <SwmPath>[shopizer/…/business/customer/](shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/customer/)</SwmPath>`**`. Logout and login processing URLs are clearly defined to manage session control effectively.

# Admin Endpoints Details

Admin endpoints include URLs such as `/admin/logon.html`, `/admin/denied.html`, and `/admin/unauthorized.html`, which are accessible without authentication to facilitate login and error handling. The login process is handled via `/admin/j_spring_security_check`, redirecting successful logins to `/admin/home.html`. Logout is managed through `/admin/j_spring_security_logout`, redirecting users back to the admin home page. Corresponding JSP pages like <SwmPath>[shopizer/…/admin/logon.jsp](shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/logon.jsp)</SwmPath> and <SwmPath>[shopizer/…/admin/unauthorized.jsp](shopizer/sm-shop/src/main/webapp/WEB-INF/views/admin/unauthorized.jsp)</SwmPath> provide the user interface for these flows.

# Shop Endpoints Details

Shop endpoints allow public access to most URLs, including customer login and registration pages such as `/shop/customer/logon.html` and `/shop/customer/registration.html`. Authenticated customers with the 'AUTH_CUSTOMER' role gain access to protected URLs under <SwmPath>[shopizer/…/business/customer/](shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/customer/)</SwmPath>`**`. Customer logout is handled via `/shop/customer/j_spring_security_logout`, redirecting to the shop's main page. JSP files like <SwmPath>[shopizer/…/products/product.jsp](shopizer/sm-shop/src/main/webapp/pages/admin/products/product.jsp)</SwmPath>, <SwmPath>[shopizer/…/categories/category.jsp](shopizer/sm-shop/src/main/webapp/pages/admin/categories/category.jsp)</SwmPath>, and <SwmPath>[shopizer/…/checkout/checkout.jsp](shopizer/sm-shop/src/main/webapp/pages/shop/common/checkout/checkout.jsp)</SwmPath> render the storefront and shopping experience. The shop also exposes REST services under <SwmPath>[shopizer/…/web/services/](shopizer/sm-shop/src/main/java/com/salesmanager/web/services/)</SwmPath>`**` with appropriate access controls.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
