---
title: Shopping Cart Total Calculation Flow
---
This document describes the flow of calculating the total cost of a shopping cart. It validates inputs, delegates detailed calculation, prepares an order summary, calculates subtotal, shipping, handling fees, and taxes, updates the cart model, and returns the total summary to ensure accurate pricing before checkout.

```mermaid
flowchart TD
  node1["Starting the Shopping Cart Total Calculation"]:::HeadingStyle
  click node1 goToHeading "Starting the Shopping Cart Total Calculation"
  node1 --> node2["Delegating Shopping Cart Total Calculation"]:::HeadingStyle
  click node2 goToHeading "Delegating Shopping Cart Total Calculation"
  node2 --> node3["Preparing Order Summary from Cart Items"]:::HeadingStyle
  click node3 goToHeading "Preparing Order Summary from Cart Items"
  node3 --> node4["Calculating Detailed Order Totals"]:::HeadingStyle
  click node4 goToHeading "Calculating Detailed Order Totals"
  node4 --> node5["Finalizing Cart Update and Returning Summary"]:::HeadingStyle
  click node5 goToHeading "Finalizing Cart Update and Returning Summary"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Starting the Shopping Cart Total Calculation

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start calculation"]
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartCalculationServiceImpl.java:95:97"
    node1 --> node2{"Are cart, line items, and store valid?"}
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartCalculationServiceImpl.java:98:100"
    node2 -->|"Yes"| node3["Delegating Shopping Cart Total Calculation"]
    
    node3 --> node4["Preparing Order Summary from Cart Items"]
    
    node4 --> node5["Finalizing Cart Update and Returning Summary"]
    
    node5 --> node6["Return order total summary"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node3 goToHeading "Delegating Shopping Cart Total Calculation"
node3:::HeadingStyle
click node4 goToHeading "Preparing Order Summary from Cart Items"
node4:::HeadingStyle
click node4b goToHeading "Calculating Detailed Order Totals"
node4b:::HeadingStyle
click node5 goToHeading "Finalizing Cart Update and Returning Summary"
node5:::HeadingStyle
```

This section describes the process of starting the shopping cart total calculation in the e-commerce platform. It ensures the validity of the cart, its items, and the store before delegating the detailed total calculation to another service. After calculation, it updates the cart model and returns the order total summary.

| Category        | Rule Name                  | Description                                                                                                                                              |
| --------------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Data validation | Input Validation           | The shopping cart, its line items, and the merchant store must be valid and not null before starting the total calculation.                              |
| Business logic  | Cart Model Update          | After the total calculation, the shopping cart model must be updated to reflect any changes resulting from the calculation before returning the summary. |
| Business logic  | Order Total Summary Output | The final output of the process is an order total summary that provides a detailed breakdown of the shopping cart totals.                                |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartCalculationServiceImpl.java" line="95">

---

Here in ShoppingCartCalculationServiceImpl.calculate, we start by checking that the cart, its items, and the store are valid. Then we call orderService.calculateShoppingCartTotal to get the detailed total summary. After that, we update the cart model to reflect any changes from the calculation before returning the summary.

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

## Delegating Shopping Cart Total Calculation

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is shopping cart provided?"}
    node1 -->|"No"| node2["Invalid shopping cart"]
    node1 -->|"Yes"| node3{"Is merchant store provided?"}
    node3 -->|"No"| node4["Invalid merchant store"]
    node3 -->|"Yes"| node5["Delegate to detailed calculation function"]
    node5 --> node6["Return order total summary"]
    node5 -->|"Failure"| node7["Calculation failed"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section describes the process of delegating the calculation of the shopping cart total in the Shopizer platform.

| Category       | Rule Name            | Description                                                                                                                     |
| -------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Delegate calculation | The calculation of the shopping cart total is delegated to a detailed calculation function that handles the actual computation. |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java" line="423">

---

In calculateShoppingCartTotal, we check the shopping cart and store are valid, then call caculateShoppingCart passing null for the customer since it's not needed here. This method handles exceptions and logs errors if calculation fails.

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

## Preparing Order Summary from Cart Items

This section prepares an order summary from the shopping cart items by creating an OrderSummary object, populating it with the cart items, and then calculating the order totals.

| Category       | Rule Name              | Description                                                                                                           |
| -------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Business logic | Order summary creation | An order summary must be created from the shopping cart items before any calculations are performed.                  |
| Business logic | Calculate order totals | After preparing the order summary, the system must calculate the order totals including prices, taxes, and discounts. |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java" line="363">

---

CaculateShoppingCart takes the shopping cart items, puts them into an OrderSummary, then calls caculateOrder to do the heavy lifting of calculating totals. It ignores the customer here since it's null.

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

## Calculating Detailed Order Totals

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start order calculation"] --> loop1["For each product in order"]
    loop1 --> node2["Calculate subtotal including additional one-time prices"]
    node2 --> node3["Accumulate subtotal"]
    node3 --> node4{"Is shipping free?"}
    node4 -->|"No"| node5["Add shipping cost to grand total"]
    node4 -->|"Yes"| node6["Add zero shipping cost"]
    node5 --> node7{"Are handling fees applicable and configured?"}
    node6 --> node7
    node7 -->|"Yes"| node8["Add handling fees to grand total"]
    node7 -->|"No"| node9["Skip handling fees"]
    node8 --> node10["Calculate taxes using taxService"]
    node9 --> node10
    node10 --> loop2["For each tax item"]
    loop2 --> node11["Add tax item price to total taxes"]
    node11 --> node12["Add total taxes to grand total"]
    node12 --> node13["Return order total summary with grand total"]

    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:166:170"
    click loop1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:184:223"
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:186:188"
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:182:188"
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:247:266"
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:261:262"
    click node6 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:264:265"
    click node7 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:269:283"
    click node8 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:279:281"
    click node9 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:283:283"
    click node10 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:287:310"
    click loop2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:292:309"
    click node11 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:303:303"
    click node12 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:310:311"
    click node13 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java:325:327"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section calculates the detailed order totals including subtotal, shipping, handling fees, taxes, and grand total for an order.

| Category       | Rule Name                    | Description                                                                                                                                          |
| -------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Subtotal calculation         | Calculate the subtotal by multiplying each product's price by its quantity and adding any additional one-time prices.                                |
| Business logic | Include additional prices    | Add additional non-default prices to the subtotal, including one-time prices, to reflect all applicable product costs.                               |
| Business logic | Shipping cost inclusion      | Add shipping cost to the grand total unless free shipping is applicable, in which case add zero shipping cost.                                       |
| Business logic | Handling fees application    | Add handling fees to the grand total only if handling fees are configured and the order's shipping summary includes handling fees greater than zero. |
| Business logic | Tax calculation and addition | Calculate taxes for the order and add each tax item to the total taxes and grand total.                                                              |
| Business logic | Grand total composition      | The grand total is the sum of subtotal, shipping cost, handling fees, and total taxes, representing the final amount payable.                        |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java" line="166">

---

In caculateOrder, we loop through each item to calculate the subtotal by multiplying price and quantity. We also check for additional prices, accumulate them by code, and add one-time prices to the subtotal.

```java
    private OrderTotalSummary caculateOrder(final OrderSummary summary, final Customer customer, final MerchantStore store, final Language language) throws Exception {

        OrderTotalSummary totalSummary = new OrderTotalSummary();
        List<OrderTotal> orderTotals = new ArrayList<OrderTotal>();
        Map<String,OrderTotal> otherPricesTotals = new HashMap<String,OrderTotal>();

        ShippingConfiguration shippingConfiguration = null;

        BigDecimal grandTotal = new BigDecimal(0);
        grandTotal.setScale(2, RoundingMode.HALF_UP);

        //price by item
        /**
         * qty * price
         * subtotal
         */
        BigDecimal subTotal = new BigDecimal(0);
        subTotal.setScale(2, RoundingMode.HALF_UP);
        for(ShoppingCartItem item : summary.getProducts()) {

            BigDecimal st = item.getItemPrice().multiply(new BigDecimal(item.getQuantity()));
            item.setSubTotal(st);
            subTotal = subTotal.add(st);
            //Other prices
            FinalPrice finalPrice = item.getFinalPrice();
            if(finalPrice!=null) {
                List<FinalPrice> otherPrices = finalPrice.getAdditionalPrices();
                if(otherPrices!=null) {
                    for(FinalPrice price : otherPrices) {
                        if(!price.isDefaultPrice()) {
                            OrderTotal itemSubTotal = otherPricesTotals.get(price.getProductPrice().getCode());

                            if(itemSubTotal==null) {
                                itemSubTotal = new OrderTotal();
                                itemSubTotal.setModule(Constants.OT_ITEM_PRICE_MODULE_CODE);
                                itemSubTotal.setText(Constants.OT_ITEM_PRICE_MODULE_CODE);
                                itemSubTotal.setTitle(Constants.OT_ITEM_PRICE_MODULE_CODE);
                                itemSubTotal.setOrderTotalCode(price.getProductPrice().getCode());
                                itemSubTotal.setOrderTotalType(OrderTotalType.PRODUCT);
                                itemSubTotal.setSortOrder(0);
                                otherPricesTotals.put(price.getProductPrice().getCode(), itemSubTotal);
                            }

                            BigDecimal orderTotalValue = itemSubTotal.getValue();
                            if(orderTotalValue==null) {
                                orderTotalValue = new BigDecimal(0);
                                orderTotalValue.setScale(2, RoundingMode.HALF_UP);
                            }

                            orderTotalValue = orderTotalValue.add(price.getFinalPrice());
                            itemSubTotal.setValue(orderTotalValue);
                            if(price.getProductPrice().getProductPriceType().name().equals(OrderValueType.ONE_TIME)) {
                                subTotal = subTotal.add(price.getFinalPrice());
                            }
                        }
                    }
                }
            }

        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java" line="228">

---

After calculating subtotal, we add it to the grand total and create an OrderTotal object for it. Then we handle shipping and handling fees similarly, using constants to label each part and add their values to the grand total.

```java
        totalSummary.setSubTotal(subTotal);
        grandTotal=grandTotal.add(subTotal);

        OrderTotal orderTotalSubTotal = new OrderTotal();
        orderTotalSubTotal.setModule(Constants.OT_SUBTOTAL_MODULE_CODE);
        orderTotalSubTotal.setOrderTotalType(OrderTotalType.SUBTOTAL);
        orderTotalSubTotal.setOrderTotalCode("order.total.subtotal");
        orderTotalSubTotal.setTitle(Constants.OT_SUBTOTAL_MODULE_CODE);
        orderTotalSubTotal.setText("order.total.subtotal");
        orderTotalSubTotal.setSortOrder(5);
        orderTotalSubTotal.setValue(subTotal);

        //TODO autowire a list of post processing modules for price calculation - drools, custom modules
        //may affect the sub total

        orderTotals.add(orderTotalSubTotal);


        //shipping
        if(summary.getShippingSummary()!=null) {


	            OrderTotal shippingSubTotal = new OrderTotal();
	            shippingSubTotal.setModule(Constants.OT_SHIPPING_MODULE_CODE);
	            shippingSubTotal.setOrderTotalType(OrderTotalType.SHIPPING);
	            shippingSubTotal.setOrderTotalCode("order.total.shipping");
	            shippingSubTotal.setTitle(Constants.OT_SHIPPING_MODULE_CODE);
	            shippingSubTotal.setText("order.total.shipping");
	            shippingSubTotal.setSortOrder(10);
	
	            orderTotals.add(shippingSubTotal);

            if(!summary.getShippingSummary().isFreeShipping()) {
                shippingSubTotal.setValue(summary.getShippingSummary().getShipping());
                grandTotal=grandTotal.add(summary.getShippingSummary().getShipping());
            } else {
                shippingSubTotal.setValue(new BigDecimal(0));
                grandTotal=grandTotal.add(new BigDecimal(0));
            }

            //check handling fees
            shippingConfiguration = shippingService.getShippingConfiguration(store);
            if(summary.getShippingSummary().getHandling()!=null && summary.getShippingSummary().getHandling().doubleValue()>0) {
                if(shippingConfiguration.getHandlingFees()!=null && shippingConfiguration.getHandlingFees().doubleValue()>0) {
                    OrderTotal handlingubTotal = new OrderTotal();
                    handlingubTotal.setModule(Constants.OT_HANDLING_MODULE_CODE);
                    handlingubTotal.setOrderTotalType(OrderTotalType.HANDLING);
                    handlingubTotal.setOrderTotalCode("order.total.handling");
                    handlingubTotal.setTitle(Constants.OT_HANDLING_MODULE_CODE);
                    handlingubTotal.setText("order.total.handling");
                    handlingubTotal.setSortOrder(12);
                    handlingubTotal.setValue(summary.getShippingSummary().getHandling());
                    orderTotals.add(handlingubTotal);
                    grandTotal=grandTotal.add(summary.getShippingSummary().getHandling());
                }
            }
        }

        //tax
        List<TaxItem> taxes = taxService.calculateTax(summary, customer, store, language);
        if(taxes!=null && taxes.size()>0) {
        	BigDecimal totalTaxes = new BigDecimal(0);
        	totalTaxes.setScale(2, RoundingMode.HALF_UP);
            int taxCount = 20;
            for(TaxItem tax : taxes) {

                OrderTotal taxLine = new OrderTotal();
                taxLine.setModule(Constants.OT_TAX_MODULE_CODE);
                taxLine.setOrderTotalType(OrderTotalType.TAX);
                taxLine.setOrderTotalCode(tax.getLabel());
                taxLine.setSortOrder(taxCount);
                taxLine.setTitle(Constants.OT_TAX_MODULE_CODE);
                taxLine.setText(tax.getLabel());
                taxLine.setValue(tax.getItemPrice());

                totalTaxes = totalTaxes.add(tax.getItemPrice());
                orderTotals.add(taxLine);
                //grandTotal=grandTotal.add(tax.getItemPrice());

                taxCount ++;

            }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/order/service/OrderServiceImpl.java" line="310">

---

Finally, we calculate taxes, add each tax as an OrderTotal, sum them into the grand total, create an OrderTotal for the grand total, and return the complete total summary.

```java
            grandTotal = grandTotal.add(totalTaxes);
            totalSummary.setTaxTotal(totalTaxes);
        }

        // grand total
        OrderTotal orderTotal = new OrderTotal();
        orderTotal.setModule(Constants.OT_TOTAL_MODULE_CODE);
        orderTotal.setOrderTotalType(OrderTotalType.TOTAL);
        orderTotal.setOrderTotalCode("order.total.total");
        orderTotal.setTitle(Constants.OT_TOTAL_MODULE_CODE);
        orderTotal.setText("order.total.total");
        orderTotal.setSortOrder(300);
        orderTotal.setValue(grandTotal);
        orderTotals.add(orderTotal);

        totalSummary.setTotal(grandTotal);
        totalSummary.setTotals(orderTotals);
        return totalSummary;

    }
```

---

</SwmSnippet>

## Finalizing Cart Update and Returning Summary

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartCalculationServiceImpl.java" line="102">

---

We just came back from the order service with the total summary. Now in ShoppingCartCalculationServiceImpl.calculate, we update the cart model to reflect any changes, then return the summary.

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
