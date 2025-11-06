---
title: Order Facade Overview
---
# Overview of Order Facade

The Order Facade serves as a service layer that simplifies interactions with the complex order processing system within the application. It provides a unified interface that consolidates multiple underlying services related to order management, product handling, customer management, pricing, shipping, and language support.

By centralizing these operations, the Order Facade abstracts the detailed logic involved in assembling order data, managing shopping cart items, and integrating customer information. This abstraction allows higher-level components, such as web controllers or APIs, to interact with orders without needing to understand the complexities of the underlying services.

# Facade Design Pattern in Order Processing

The Facade pattern is a structural design pattern that offers a simplified interface to a complex system of classes or services. In the context of order processing, it reduces dependencies and complexity for client code by providing a higher-level interface that encapsulates multiple domain services.

Using the Facade promotes separation of concerns by centralizing business logic related to orders into one service layer. This design reduces coupling between the web layer and business services, making the system easier to maintain, test, and extend.

# Implementation of Order Facade

The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="87:8:8" line-data="public class OrderFacadeImpl implements OrderFacade {">`OrderFacade`</SwmToken> interface defines key operations such as initializing orders, calculating order totals, processing orders, refreshing order data, validating orders, and retrieving shipping information. Its implementation, typically named <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="87:4:4" line-data="public class OrderFacadeImpl implements OrderFacade {">`OrderFacadeImpl`</SwmToken>, injects various services including <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="44:14:14" line-data="import com.salesmanager.core.business.order.service.OrderService;">`OrderService`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="24:16:16" line-data="import com.salesmanager.core.business.catalog.product.service.ProductService;">`ProductService`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="30:14:14" line-data="import com.salesmanager.core.business.customer.service.CustomerService;">`CustomerService`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="23:16:16" line-data="import com.salesmanager.core.business.catalog.product.service.PricingService;">`PricingService`</SwmToken>, and <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="58:14:14" line-data="import com.salesmanager.core.business.shipping.service.ShippingService;">`ShippingService`</SwmToken>.

<SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="87:4:4" line-data="public class OrderFacadeImpl implements OrderFacade {">`OrderFacadeImpl`</SwmToken> orchestrates these services to fulfill the required operations, abstracting the complexity from the caller. This allows clients to interact with a single interface rather than managing multiple service dependencies.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" line="127">

---

One important method is <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="128:5:5" line-data="	public ShopOrder initializeOrder(MerchantStore store, Customer customer,">`initializeOrder`</SwmToken>, which creates a new <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="128:3:3" line-data="	public ShopOrder initializeOrder(MerchantStore store, Customer customer,">`ShopOrder`</SwmToken> object based on the current shopping cart, customer, and store state. It sets the initial order status and associates the customer and shopping cart items with the order. This method encapsulates the complexity of assembling order data, preparing it for further processing or display.

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

Another critical operation is <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="155:5:5" line-data="	public OrderTotalSummary calculateOrderTotal(MerchantStore store,">`calculateOrderTotal`</SwmToken>, which has two overloaded versions accepting either a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="156:1:1" line-data="			ShopOrder order, Language language) throws Exception {">`ShopOrder`</SwmToken> or a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="167:1:1" line-data="			PersistableOrder order, Language language) throws Exception {">`PersistableOrder`</SwmToken>. These methods compute the total cost of the order, including pricing, taxes, and discounts. They interact with customer data and shopping cart items to produce an <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/order/facade/OrderFacadeImpl.java" pos="155:3:3" line-data="	public OrderTotalSummary calculateOrderTotal(MerchantStore store,">`OrderTotalSummary`</SwmToken>, which updates the order totals. This encapsulates pricing logic and ensures consistency in total calculations.

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

# Benefits of Using Order Facade

By consolidating all order-related operations into a single service layer, the Order Facade reduces coupling between web controllers and business services. This simplification facilitates easier testing and allows modifications or extensions to order processing logic without impacting other parts of the application.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
