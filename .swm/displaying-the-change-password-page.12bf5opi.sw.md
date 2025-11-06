---
title: Displaying the Change Password Page
---
This document describes how the change password page is displayed for a customer. When requested, the system uses the current store context to render a branded password change page with an empty form, ensuring a secure and consistent experience.

# Rendering the Change Password Page

This section governs how the change password page is rendered for a customer, ensuring the correct store theme is used and the password change model is properly initialized.

| Category        | Rule Name                           | Description                                                                                                                                                                                                                                                                                                                                                         |
| --------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Data validation | Session-based store context         | The merchant store must be retrieved from the session using a predefined constant key to ensure the correct context for rendering the page.                                                                                                                                                                                                                         |
| Business logic  | Store-specific password page theme  | The change password page must be rendered using the template associated with the current merchant store's theme.                                                                                                                                                                                                                                                    |
| Business logic  | Fresh password model initialization | A new <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" pos="169:1:1" line-data="		CustomerPassword customerPassword = new CustomerPassword();">`CustomerPassword`</SwmToken> model must be provided to the view to ensure the password change form is initialized empty for the customer. |

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" line="164">

---

DisplayCustomerChangePassword kicks off the flow for rendering the change password page. It grabs the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" pos="167:1:1" line-data="	    MerchantStore store = getSessionAttribute(Constants.MERCHANT_STORE, request);">`MerchantStore`</SwmToken> from the session (using a constant key), sets up a fresh <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" pos="169:1:1" line-data="		CustomerPassword customerPassword = new CustomerPassword();">`CustomerPassword`</SwmToken> model, and then builds the template path by combining a base constant with the store's template name. This lets each store use its own theme for the password page. The function returns the template path string for view rendering.

```java
	public String displayCustomerChangePassword(Model model, HttpServletRequest request, HttpServletResponse response) throws Exception {
		

	    MerchantStore store = getSessionAttribute(Constants.MERCHANT_STORE, request);

		CustomerPassword customerPassword = new CustomerPassword();
		model.addAttribute("password", customerPassword);
		
		/** template **/
		StringBuilder template = new StringBuilder().append(ControllerConstants.Tiles.Customer.changePassword).append(".").append(store.getStoreTemplate());

		return template.toString();
		
	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
