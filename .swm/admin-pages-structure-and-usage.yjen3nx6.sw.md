---
title: Admin Pages Structure and Usage
---
# Overview of Admin Pages

The Admin section consists of a comprehensive collection of JSP files that form the user interface for managing the e-commerce platform. These pages enable administrators to control various aspects such as tax settings, shipping options, user profiles, product details, payment methods, orders, merchants, customers, content, system configuration, categories, and components.

# Structure and Organization

The Admin pages are organized into subdirectories within the admin folder, each corresponding to a specific administrative domain. For example, there are dedicated folders for tax, shipping, profiles, products, payments, orders, merchants, customers, content, configuration, categories, and components. This modular structure promotes separation of concerns, allowing administrators to focus on particular areas without distraction from unrelated functionalities.

# Functionality and User Interface

Each JSP page within these subdirectories serves as a front-end view that supports CRUD (Create, Read, Update, Delete) operations and configuration management for its domain. The pages typically include grid headers, detailed views, and lists to facilitate efficient navigation and management of data. This design ensures that administrators can easily browse, edit, and update information relevant to their tasks.

# Usage Example: Product and Order Management

For instance, the 'products' subdirectory contains JSP files that allow administrators to manage product details, including pricing and descriptions. Similarly, the 'orders' subdirectory provides interfaces for handling order processing, including viewing order details and updating statuses. These pages enable administrators to maintain the e-commerce platform effectively by providing intuitive and domain-specific management tools.

# Admin Endpoints and Their Role

Admin endpoints are URL paths that handle administrative functions by processing form submissions from the JSP pages. For example, the endpoint `/admin/products/price/save.html` is used to save or update product pricing information. This is linked to a form in the JSP file <SwmPath>[shopizer/…/products/price.jsp](shopizer/sm-shop/src/main/webapp/pages/admin/products/price.jsp)</SwmPath>, which includes fields for price, special price, and validity dates. When the form is submitted, a POST request is sent to this endpoint to persist the pricing data.

Similarly, the endpoint `/admin/orders/save.html` processes updates to order details. The corresponding JSP file <SwmPath>[shopizer/…/orders/order.jsp](shopizer/sm-shop/src/main/webapp/pages/admin/orders/order.jsp)</SwmPath> contains a form that allows administrators to modify billing and shipping addresses, order status, and transaction information. Submitting this form sends a POST request to the endpoint, which updates the order data in the system.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
