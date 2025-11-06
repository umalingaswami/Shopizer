---
title: Webapp Directory Structure and Purpose
---
# Overview of the Webapp Directory

The webapp directory in Shop Web contains all the essential web resources and configuration files required for the user interface and deployment of the web application. This includes JSP files that define page layouts, configuration files for servlet mappings, and tag libraries that provide custom JSP tags used throughout the application.

# Structure and Organization of Web Resources

Within the webapp directory, developers organize static assets, dynamic templates, and configuration files to maintain a clean and modular structure. For example, JSP files such as <SwmPath>[shopizer/…/common/adminTabs.jsp](shopizer/sm-shop/src/main/webapp/common/adminTabs.jsp)</SwmPath> and <SwmPath>[shopizer/…/common/adminLinks.jsp](shopizer/sm-shop/src/main/webapp/common/adminLinks.jsp)</SwmPath> define the structure and layout of the administration interface, enabling a consistent user experience across the admin pages.

A key subdirectory under webapp is the `common` folder, which holds shared JSP fragments and resources. These reusable components promote maintainability and consistency by allowing different parts of the web interface to share common elements without duplication.

# Configuration Files in WEB-INF

The `WEB-INF` folder inside the webapp directory contains critical configuration files necessary for the web application's operation. Among these, the <SwmPath>[shopizer/…/WEB-INF/web.xml](shopizer/sm-shop/src/main/webapp/WEB-INF/web.xml)</SwmPath> file plays a central role by defining servlet mappings, filter configurations, and other deployment settings that control how the application handles requests.

# Custom Tag Libraries

To simplify JSP development and encapsulate reusable logic, the webapp directory includes tag library descriptor files (TLDs) such as <SwmPath>[shopizer/…/WEB-INF/shopizer-tags.tld](shopizer/sm-shop/src/main/webapp/WEB-INF/shopizer-tags.tld)</SwmPath> and <SwmPath>[shopizer/…/WEB-INF/shopizer-functions.tld](shopizer/sm-shop/src/main/webapp/WEB-INF/shopizer-functions.tld)</SwmPath>. These files define custom JSP tags that are leveraged throughout the application to enhance code readability and reduce redundancy.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
