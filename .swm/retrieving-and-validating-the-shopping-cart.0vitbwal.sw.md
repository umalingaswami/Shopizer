---
title: Retrieving and Validating the Shopping Cart
---
This document describes how a customer's shopping cart is retrieved and validated to ensure it contains only current and valid items. The flow receives a customer request and returns an updated shopping cart or null if the cart is obsolete.

```mermaid
flowchart TD
  node1["Fetching and Validating the Customer's Cart"]:::HeadingStyle
  click node1 goToHeading "Fetching and Validating the Customer's Cart"
  node1 --> node2["Retrieving and Preparing the Cart from Storage"]:::HeadingStyle
  click node2 goToHeading "Retrieving and Preparing the Cart from Storage"
  node2 --> node3{"Are cart items valid?
(Refreshing and Cleaning Up Cart Items)"}:::HeadingStyle
  click node3 goToHeading "Refreshing and Cleaning Up Cart Items"
  node3 -->|"Yes"| node4["Return updated shopping cart
(Finalizing the Cart After Retrieval and Population)"]:::HeadingStyle
  click node4 goToHeading "Finalizing the Cart After Retrieval and Population"
  node3 -->|"No"| node5["Cart marked obsolete, return null
(Finalizing the Cart After Retrieval and Population)"]:::HeadingStyle
  click node5 goToHeading "Finalizing the Cart After Retrieval and Population"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Fetching and Validating the Customer's Cart

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="68">

---

In `getShoppingCart`, we start by pulling the cart for the customer using getByCustomer, since everything else depends on having the right cart for this user.

```java
	public ShoppingCart getShoppingCart(final Customer customer) throws ServiceException {

		try {

			ShoppingCart shoppingCart = shoppingCartDao.getByCustomer(customer);
```

---

</SwmSnippet>

## Retrieving and Preparing the Cart from Storage

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start: Retrieve shopping cart for customer"]
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:209:210"
    node1 --> node2["Get shopping cart from database"]
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:212:212"
    node2 --> node3{"Does the customer have a shopping cart?"}
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:213:215"
    node3 -->|"No"| node4["Return null"]
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:214:214"
    node3 -->|"Yes"| node5["Populate shopping cart"]
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:216:216"
    node5 --> node6["Return populated shopping cart"]
    click node6 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:216:216"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="209">

---

`getByCustomer` fetches the cart for the customer from the database. If there's no cart, it returns null right away. If a cart exists, it calls populateShoppingCart to update and clean up the cart before returning it.

```java
	public ShoppingCart getByCustomer(final Customer customer) throws ServiceException {

		try {
			ShoppingCart shoppingCart = shoppingCartDao.getByCustomer(customer);
			if(shoppingCart==null) {
				return null;
			}
			return populateShoppingCart(shoppingCart);


		} catch (Exception e) {
			throw new ServiceException(e);
		}
	}
```

---

</SwmSnippet>

## Refreshing and Cleaning Up Cart Items

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="225">

---

In `populateShoppingCart`, we loop through each item in the cart and call populateItem to update its state and pricing. This step also checks if items are obsolete and flags the cart as obsolete if needed. It's not just about filling in data—it's about cleaning up and validating the cart before it's used.

```java
	private ShoppingCart populateShoppingCart(final ShoppingCart shoppingCart) throws Exception {

		try {

			boolean cartIsObsolete = true;
			if(shoppingCart!=null) {

				Set<ShoppingCartItem> items = shoppingCart.getLineItems();
				if(items==null || items.size()==0) {
					shoppingCart.setObsolete(true);
					return shoppingCart;

				}

				//Set<ShoppingCartItem> shoppingCartItems = new HashSet<ShoppingCartItem>();
				for(ShoppingCartItem item : items) {
					LOGGER.debug("Populate item " + item.getId());
					populateItem(item);
					LOGGER.debug("Obsolete item ? " + item.isObsolete());
					if(item.isObsolete()) {
					} else {
						cartIsObsolete = false;
					}
				}

```

---

</SwmSnippet>

### Updating Cart Item Details and Pricing

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Lookup product for cart item"]
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:319:320"
    node1 --> node2{"Is product found?"}
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:322:325"
    node2 -->|"No"| node3["Mark item as obsolete"]
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:323:324"
    node2 -->|"Yes"| node4["Attach product to item"]
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:328:328"
    node4 --> node5{"Is product virtual?"}
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:330:332"
    node5 -->|"Yes"| node6["Mark item as virtual"]
    click node6 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:331:331"
    node5 -->|"No"| node8{"Are there attributes to match?"}
    node6 --> node8
    node4 --> node8
    click node8 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:337:350"
    subgraph loop1["For each attribute in cart item"]
        node8 -->|"Yes"| node9["Match attribute to product attribute"]
        click node9 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:338:348"
        node9 --> node8
    end
    node8 -->|"No attributes"| node10["Calculate item price"]
    click node10 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:354:356"
    node10 --> node11["Set subtotal for item"]
    click node11 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:360:361"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="315">

---

In `populateItem`, we fetch the product for the cart item. If it's missing, we mark the item obsolete and stop. Otherwise, we update the item with the product, match up attributes, recalculate the price, and set the subtotal. This keeps the item in sync with the latest product data.

```java
	private void populateItem(final ShoppingCartItem item) throws Exception {

		Product product = null;

		Long productId = item.getProductId();
		product = productService.getById(productId);

		if(product==null) {
			item.setObsolete(true);
			return;
		}


		item.setProduct(product);
		
		if(product.isProductVirtual()) {
			item.setProductVirtual(true);
		}

		Set<ShoppingCartAttributeItem> attributes = item.getAttributes();
		Set<ProductAttribute> productAttributes = product.getAttributes();
		List<ProductAttribute> attributesList = new ArrayList<ProductAttribute>();
		if(productAttributes!=null && productAttributes.size()>0 && attributes!=null && attributes.size()>0) {
			for(ShoppingCartAttributeItem attribute : attributes) {
				long attributeId = attribute.getProductAttributeId().longValue();
				for(ProductAttribute productAttribute : productAttributes) {

					if(productAttribute.getId().longValue()==attributeId) {
						attribute.setProductAttribute(productAttribute);
						attributesList.add(productAttribute);
						break;
					}

				}

			}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="353">

---

After `populateItem`, the item has updated pricing and subtotal, or it's marked obsolete if the product is missing.

```java
		//set item price
		FinalPrice price = pricingService.calculateProductPrice(product, attributesList);
		item.setItemPrice(price.getFinalPrice());
		item.setFinalPrice(price);



		BigDecimal subTotal = item.getItemPrice().multiply(new BigDecimal(item.getQuantity().intValue()));
		item.setSubTotal(subTotal);


	}
```

---

</SwmSnippet>

### Filtering and Updating Cart Items After Population

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start: Process shopping cart items"]
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:250:253"
    subgraph loop1["For each item in cart"]
        node2{"Is item obsolete?"}
        click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:254:258"
        node2 -->|"No"| node3["Add item to refreshed set"]
        click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:255:255"
        node2 -->|"Yes"| node4["Mark cart for refresh"]
        click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:257:258"
    end
    loop1 --> node5{"Was any item removed?"}
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:261:261"
    node5 -->|"Yes"| node6["Update cart with refreshed items"]
    click node6 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:262:263"
    node6 --> node7{"Is cart obsolete?"}
    click node7 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:266:267"
    node5 -->|"No"| node7
    node7 -->|"Yes"| node8["Mark cart as obsolete"]
    click node8 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:267:268"
    node7 -->|"No"| node9["Return shopping cart"]
    click node9 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:269:269"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="250">

---

Back in `populateShoppingCart`, after running populateItem on each item, we filter out obsolete items and flag if the cart needs to be updated. Only valid items are kept for the next step.

```java
				//shoppingCart.setLineItems(shoppingCartItems);
				boolean refreshCart = false;
                Set<ShoppingCartItem> refreshedItems = new HashSet<ShoppingCartItem>();
                for(ShoppingCartItem item : items) {
                	if(!item.isObsolete()) {
                		refreshedItems.add(item);
                	} else {
                		refreshCart = true;
                	}
                }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="261">

---

After cleaning up items in `populateShoppingCart`, we update the cart if any items were removed, mark it obsolete if needed, and return the updated cart. If everything's obsolete, the cart is flagged as such.

```java
                if(refreshCart) {
                	shoppingCart.setLineItems(refreshedItems);
                	update(shoppingCart);
                }

				if(cartIsObsolete) {
					shoppingCart.setObsolete(true);
				}
				return shoppingCart;
			}

		} catch (Exception e) {
			throw new ServiceException(e);
		}

		return shoppingCart;

	}
```

---

</SwmSnippet>

## Finalizing the Cart After Retrieval and Population

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="73">

---

Back in `getShoppingCart`, after getting the cart from getByCustomer, we call populateShoppingCart to make sure all the items are current and the cart is cleaned up before we do anything else with it.

```java
			populateShoppingCart(shoppingCart);
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="74">

---

After `populateShoppingCart` in `getShoppingCart`, we check if the cart is obsolete. If it is, we delete it and return null; otherwise, we just return the cart. This keeps users from seeing or using dead carts.

```java
			if(shoppingCart!=null && shoppingCart.isObsolete()) {
				delete(shoppingCart);
				return null;
			} else {
				return shoppingCart;
			}


		} catch (Exception e) {
			throw new ServiceException(e);
		}

	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
