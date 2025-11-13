---
title: Calculating shopping cart total flow
---
This document describes the flow of calculating the total amount for a shopping cart. It validates the cart and store data, delegates the calculation to a service that processes cart items and computes the order summary, then updates the cart model and returns the total summary. This flow supports the shopping experience by providing accurate pricing.

# Starting the Shopping Cart Total Calculation

This section initiates the calculation of the shopping cart total by validating inputs and delegating the calculation to the order service.

| Category        | Rule Name            | Description                                                           |
| --------------- | -------------------- | --------------------------------------------------------------------- |
| Data validation | Non-null cart        | The shopping cart must not be null to proceed with total calculation. |
| Data validation | Cart must have items | The shopping cart must contain line items to calculate a total.       |
| Data validation | Valid merchant store | The merchant store information must be provided and not null.         |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartCalculationServiceImpl.java" line="95">

---

The flow starts by checking inputs and then hands off the total calculation to the order service, which does the heavy lifting.

```java
    public OrderTotalSummary calculate( final ShoppingCart cartModel , final MerchantStore store, final Language language ) throws ServiceException
    {

        Validate.notNull(cartModel,"cart cannot be null");
        Validate.notNull(cartModel.getLineItems(),"Cart should have line items.");
        Validate.notNull(store,"MerchantStore cannot be null");
        OrderTotalSummary orderTotalSummary=orderService.calculateShoppingCartTotal( cartModel, store, language );
```

---

</SwmSnippet>

## Delegating Shopping Cart Total Calculation to Detailed Processor

This section describes how the shopping cart total is calculated by delegating to a detailed processor that converts shopping cart items and handles errors.

| Category       | Rule Name                 | Description                                                                                                 |
| -------------- | ------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Business logic | Customer data optionality | The shopping cart total calculation does not use customer-specific data when the customer argument is null. |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java" line="423">

---

Here, the method validates inputs and then calls caculateShoppingCart with a null customer argument, indicating no customer-specific data is used. It wraps this call in a try-catch to handle errors gracefully.

```java
    public OrderTotalSummary calculateShoppingCartTotal(
                                                        final ShoppingCart shoppingCart, final MerchantStore store, final Language language)
                                                                        throws ServiceException {
        Validate.notNull(shoppingCart,"Order summary cannot be null");
        Validate.notNull(store,"MerchantStore cannot be null");

        try {
            return caculateShoppingCart(shoppingCart, null, store, language);
        } catch (Exception e) {
            LOGGER.error( "Error while calculating shopping cart total" +e );
            throw new ServiceException(e);
        }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java" line="363">

---

Here, the method converts the shopping cart's line items from a set to a list to fit the OrderSummary's expected format. Then it sets these products on the summary and calls the main order calculation method.

```java
    private OrderTotalSummary caculateShoppingCart( final ShoppingCart shoppingCart, final Customer customer, final MerchantStore store, final Language language) throws Exception {

        
    	
    	
    	
    	OrderSummary orderSummary = new OrderSummary();
    	
    	List<ShoppingCartItem> itemsSet = new ArrayList<ShoppingCartItem>(shoppingCart.getLineItems());
    	orderSummary.setProducts(itemsSet);
    	
    	
    	return this.caculateOrder(orderSummary, customer, store, language);

    }
```

---

</SwmSnippet>

## Finalizing Shopping Cart Calculation with Cart Update

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartCalculationServiceImpl.java" line="102">

---

We just got back from the order service with the total summary. Now, the cart model is updated to reflect any changes from the calculation, and then the summary is returned.

```java
        updateCartModel(cartModel);
        return orderTotalSummary;


    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
