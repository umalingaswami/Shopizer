---
title: Order Facade Overview
---
# What is Order Facade

The Order Facade acts as a service layer that simplifies interactions with the complex order processing system within the application. It provides a unified and streamlined interface to various underlying services involved in order management, such as product handling, customer management, pricing, and shipping.

By abstracting the detailed operations, the Order Facade coordinates calls to multiple specialized services, reducing complexity for clients that need to perform order-related tasks. This design encapsulates business logic and ensures consistency across order processing workflows.

# Facade Design Pattern Overview

The Facade is a structural design pattern that offers a simplified interface to a complex subsystem. It hides the internal complexities and provides a single entry point for clients to interact with the subsystem, promoting loose coupling and easier maintenance.

# Purpose and Benefits of Using Order Facade

Using the Order Facade reduces dependencies on the intricate inner workings of the order processing system. It centralizes business logic and interactions with multiple services, making the system easier to use, maintain, and extend.

# How Order Facade Works in the Codebase

The Order Facade exposes high-level methods that internally coordinate calls to various underlying services. For example, it handles initializing orders, calculating order totals, processing orders, refreshing order data, validating orders, and generating shipping quotes by delegating these responsibilities to specialized services.

# Implementation Details

In the order processing module, the Facade pattern is implemented through the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="87:8:8" line-data="public class OrderFacadeImpl implements OrderFacade {">`OrderFacade`</SwmToken> interface and its implementation class <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="87:4:4" line-data="public class OrderFacadeImpl implements OrderFacade {">`OrderFacadeImpl`</SwmToken>. This implementation integrates services such as <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="44:14:14" line-data="import com.salesmanager.core.business.order.service.OrderService;">`OrderService`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="24:16:16" line-data="import com.salesmanager.core.business.catalog.product.service.ProductService;">`ProductService`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="30:14:14" line-data="import com.salesmanager.core.business.customer.service.CustomerService;">`CustomerService`</SwmToken>, and <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="58:14:14" line-data="import com.salesmanager.core.business.shipping.service.ShippingService;">`ShippingService`</SwmToken> to provide a cohesive interface for order-related operations.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="127">

---

For instance, the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="128:5:5" line-data="	public ShopOrder initializeOrder(MerchantStore store, Customer customer,">`initializeOrder`</SwmToken> method in <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="87:4:4" line-data="public class OrderFacadeImpl implements OrderFacade {">`OrderFacadeImpl`</SwmToken> creates a new order by coordinating data from the customer, shopping cart, and language services. Similarly, the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="155:5:5" line-data="	public OrderTotalSummary calculateOrderTotal(MerchantStore store,">`calculateOrderTotal`</SwmToken> method computes the total cost of an order by interacting with pricing and product services. These methods encapsulate complex operations behind simple method calls, making it easier for clients to work with orders.

```java
	@Override
	public ShopOrder initializeOrder(MerchantStore store, Customer customer,
			ShoppingCart shoppingCart, Language language) throws Exception {

		//assert not null shopping cart items
		
		ShopOrder order = new ShopOrder();
		
		OrderStatus orderStatus = OrderStatus.ORDERED;
		order.setOrderStatus(orderStatus);
		
		if(customer==null) {
				customer = this.initEmptyCustomer(store);
		}
		
		PersistableCustomer persistableCustomer = persistableCustomer(customer, store, language);
		order.setCustomer(persistableCustomer);

		//keep list of shopping cart items for core price calculation
		List<ShoppingCartItem> items = new ArrayList<ShoppingCartItem>(shoppingCart.getLineItems());
		order.setShoppingCartItems(items);
		
		return order;
	}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="154">

---

The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="128:5:5" line-data="	public ShopOrder initializeOrder(MerchantStore store, Customer customer,">`initializeOrder`</SwmToken> method creates a new <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="156:1:1" line-data="			ShopOrder order, Language language) throws Exception {">`ShopOrder`</SwmToken> instance based on the provided <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="155:7:7" line-data="	public OrderTotalSummary calculateOrderTotal(MerchantStore store,">`MerchantStore`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="159:1:1" line-data="		Customer customer = customerFacade.getCustomerModel(order.getCustomer(), store, language);">`Customer`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="129:1:1" line-data="			ShoppingCart shoppingCart, Language language) throws Exception {">`ShoppingCart`</SwmToken>, and <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="156:6:6" line-data="			ShopOrder order, Language language) throws Exception {">`Language`</SwmToken>. It sets the initial order status and associates the customer and shopping cart items with the order, preparing it for further processing.

```java
	@Override
	public OrderTotalSummary calculateOrderTotal(MerchantStore store,
			ShopOrder order, Language language) throws Exception {
		

		Customer customer = customerFacade.getCustomerModel(order.getCustomer(), store, language);
		OrderTotalSummary summary = this.calculateOrderTotal(store, customer, order, language);
		this.setOrderTotals(order, summary);
		return summary;
	}

	@Override
	public OrderTotalSummary calculateOrderTotal(MerchantStore store,
			PersistableOrder order, Language language) throws Exception {
	
		List<PersistableOrderProduct> orderProducts = order.getOrderProductItems();
		
		ShoppingCartItemPopulator populator = new ShoppingCartItemPopulator();
		populator.setProductAttributeService(productAttributeService);
		populator.setProductService(productService);
		populator.setShoppingCartService(shoppingCartService);
		
		List<ShoppingCartItem> items = new ArrayList<ShoppingCartItem>();
		for(PersistableOrderProduct orderProduct : orderProducts) {
			ShoppingCartItem item = populator.populate(orderProduct, new ShoppingCartItem(), store, language);
			items.add(item);
		}
		

		Customer customer = customer(order.getCustomer(), store, language);
		
		OrderTotalSummary summary = this.calculateOrderTotal(store, customer, order, language);

		return summary;
	}
```

---

</SwmSnippet>

There are two overloaded <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="155:5:5" line-data="	public OrderTotalSummary calculateOrderTotal(MerchantStore store,">`calculateOrderTotal`</SwmToken> methods: one accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="128:3:3" line-data="	public ShopOrder initializeOrder(MerchantStore store, Customer customer,">`ShopOrder`</SwmToken> for website usage, and the other accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="167:1:1" line-data="			PersistableOrder order, Language language) throws Exception {">`PersistableOrder`</SwmToken> for API usage. Both methods use customer and product services to compute pricing, taxes, and discounts, returning an <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="155:3:3" line-data="	public OrderTotalSummary calculateOrderTotal(MerchantStore store,">`OrderTotalSummary`</SwmToken> that details the financial aspects of the order.

# Summary

The Order Facade centralizes and simplifies order processing by providing a unified interface to multiple underlying services. It encapsulates complex business logic, promotes loose coupling, and makes order-related operations more accessible and maintainable within the codebase.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
