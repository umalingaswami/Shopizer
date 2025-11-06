---
title: 'OrderFacadeImpl: Shopizer Order Management Service Implementation'
---
# Introduction

This document walks through the main ideas behind the order management implementation in <SwmPath>[shopizer/…/facade/OrderFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java)</SwmPath>. The focus is on why certain design choices were made and how the main functions fit together.

We will cover:

1. Why the order initialization process is structured as it is.
2. How order total calculation is handled and why it uses the current approach.
3. Why order processing is split into several steps and how payment handling is integrated.
4. How validation is performed and why it is so granular.
5. How readable order lists and details are constructed for API/UI consumption.

# Order initialization

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="131">

---

The initialization of an order is designed to ensure that all required entities are present and correctly linked before any further processing. If a customer is not provided, a default empty customer is created using store defaults. The order status is set immediately, and shopping cart items are attached for downstream calculations.

```java
		//assert not null shopping cart items
		
		ShopOrder order = new ShopOrder();
		
		OrderStatus orderStatus = OrderStatus.ORDERED;
		order.setOrderStatus(orderStatus);
		
		if(customer==null) {
				customer = this.initEmptyCustomer(store);
		}
```

---

</SwmSnippet>

This approach avoids null pointer issues and ensures that every order starts with a consistent structure, making later steps (like total calculation and validation) predictable.

# Order total calculation

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="176">

---

Order total calculation is separated into multiple layers to allow for flexibility in handling different order types and to keep pricing logic isolated. The calculation first converts the order's product items into shopping cart items, then delegates to the core pricing service.

```java
		List<ShoppingCartItem> items = new ArrayList<ShoppingCartItem>();
		for(PersistableOrderProduct orderProduct : orderProducts) {
			ShoppingCartItem item = populator.populate(orderProduct, new ShoppingCartItem(), store, language);
			items.add(item);
		}
```

---

</SwmSnippet>

This separation means that changes to pricing logic or product attributes can be made independently of the order facade, and it also makes it easier to support new order types in the future.

# Order processing and payment handling

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="267">

---

Order processing is split into several steps to allow for different payment methods and to ensure that all order details are set before committing to the database. The process first checks if shipping and billing addresses should be unified, then populates the order model with all necessary details, including payment and shipping information.

```java
	private Order processOrderModel(ShopOrder order, Customer customer, Transaction transaction, MerchantStore store,
			Language language) throws ServiceException {
		
		try {
			
			if(order.isShipToBillingAdress()) {//customer shipping is billing
				PersistableCustomer orderCustomer = order.getCustomer();
				Address billing = orderCustomer.getBilling();
				orderCustomer.setDelivery(billing);
			}
```

---

</SwmSnippet>

Payment handling is modular: credit card payments are masked and validated, PayPal payments require a transaction, and other payment types can be added with minimal changes. This design keeps payment logic isolated and makes it easier to add new payment modules.

# Validation logic

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="565">

---

Validation is performed in a granular way, checking each required field and adding errors to the binding result and messages map. This ensures that the UI can provide detailed feedback to users and that only valid orders are processed.

```java
			//Language language = (Language)request.getAttribute("LANGUAGE");

			//validate order shipping and billing
			if(StringUtils.isBlank(order.getCustomer().getBilling().getFirstName())) {
				FieldError error = new FieldError("customer.billing.firstName","customer.billing.firstName",messages.getMessage("NotEmpty.customer.firstName", locale));
            	bindingResult.addError(error);
            	messagesResult.put("customer.billing.firstName",messages.getMessage("NotEmpty.customer.firstName", locale));
			}
```

---

</SwmSnippet>

Granular validation is important for e-commerce because missing or invalid data can lead to failed payments, shipping errors, or compliance issues. By validating each field, the system can catch problems early and avoid downstream failures.

# Readable order list and details

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="798">

---

Readable order lists and details are constructed by populating DTOs with all necessary information for API or UI consumption. This includes customer details, order products, and totals. The conversion is handled by dedicated populator classes, which makes it easy to change the output format without touching core business logic.

```java
     private ReadableOrderList populateOrderList(final OrderList orderList,final MerchantStore store, final Language language){
        List<Order> orders = orderList.getOrders();
        ReadableOrderList returnList = new ReadableOrderList();
        if(CollectionUtils.isEmpty( orders)){
            LOGGER.info( "Order list if empty..Returning empty list" );
            returnList.setTotal(0);
            returnList.setMessage("No results for store code " + store);
            return null;
        }
```

---

</SwmSnippet>

This separation between business models and readable DTOs allows for flexible API responses and UI rendering, and makes it easier to support multiple frontends or integrations.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
