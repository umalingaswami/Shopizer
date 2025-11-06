---
title: Retrieving and Validating the Customer's Shopping Cart
---
This document outlines how the system retrieves and validates a customer's shopping cart. When a customer interacts with the system, their cart is located and checked for obsolete or invalid items. If the cart is empty or contains only obsolete items, it is removed and not returned. Otherwise, the customer receives an up-to-date cart ready for further actions.

```mermaid
flowchart TD
  node1["Fetching the Customer's Shopping Cart"]:::HeadingStyle
  click node1 goToHeading "Fetching the Customer's Shopping Cart"
  node1 --> node2{"Does the customer have a cart?"}
  node2 -->|"No"| node5["Return null"]
  node2 -->|"Yes"| node3["Validating and Refreshing Cart Items"]:::HeadingStyle
  click node3 goToHeading "Validating and Refreshing Cart Items"
  node3 --> node4{"Is the cart empty or obsolete after validation?"}
  node4 -->|"Yes"| node5
  node4 -->|"No"| node6["Return updated cart"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Fetching the Customer's Shopping Cart

This section ensures that when a customer interacts with the system, their existing shopping cart is accurately retrieved and made available for further actions such as viewing, updating, or checking out.

| Category        | Rule Name                 | Description                                                                                                                          |
| --------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Data validation | Cart ownership validation | The shopping cart returned must only contain items associated with the specific customer and not include items from other customers. |
| Business logic  | Retrieve existing cart    | If a customer exists in the system, their shopping cart must be retrieved from the database using their unique identifier.           |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="68">

---

In `getShoppingCart`, we kick things off by loading the shopping cart for the given customer from the database. We need to call `getByCustomer` next because we can't do anything else until we know what (if any) cart the customer already has.

```java
	public ShoppingCart getShoppingCart(final Customer customer) throws ServiceException {

		try {

			ShoppingCart shoppingCart = shoppingCartDao.getByCustomer(customer);
```

---

</SwmSnippet>

## Retrieving the Cart from the Database

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Retrieve shopping cart for given customer"] --> node2{"Does shopping cart exist?"}
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:212:212"
    node2 -->|"No"| node3["Return null"]
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:213:215"
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:214:214"
    node2 -->|"Yes"| node4["Populate shopping cart with details"]
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:216:216"
    node4 --> node5["Return populated shopping cart"]
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:216:216"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section governs the retrieval of a customer's shopping cart from the database, ensuring that only valid and up-to-date carts are returned, or null if no cart exists.

| Category       | Rule Name             | Description                                                                                                                                          |
| -------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Null for missing cart | If a shopping cart does not exist for the given customer, the system must return a null value to indicate the absence of a cart.                     |
| Business logic | Populate cart details | If a shopping cart exists for the customer, the system must ensure the cart is populated with up-to-date and valid item details before returning it. |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="209">

---

`getByCustomer` loads the cart for the customer from the DB. If there's no cart, it returns null. Otherwise, it calls `populateShoppingCart` to make sure the cart's items are up-to-date and valid before returning it.

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

## Validating and Refreshing Cart Items

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Check if cart is empty or has no items"] --> node2{"Is cart empty or has no items?"}
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:230:233"
    node2 -->|"Yes"| node3["Mark cart as obsolete and return"]
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:233:235"
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:234:235"
    node2 -->|"No"| node4["Populate and check each item"]
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:240:248"
    subgraph loop1["For each item in cart"]
        node4 --> node5["Populate item and check obsolete status"]
        click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:242:247"
    end
    node4 --> node6["Refresh items and update cart if needed"]
    click node6 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:253:264"
    subgraph loop2["For each item in cart"]
        node6 --> node7{"Is item obsolete?"}
        click node7 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:254:258"
        node7 -->|"No"| node8["Add item to refreshed set"]
        click node8 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:255:256"
        node7 -->|"Yes"| node9["Set refreshCart flag"]
        click node9 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:257:258"
    end
    node6 --> node10{"Should cart be refreshed?"}
    click node10 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:261:264"
    node10 -->|"Yes"| node11["Update cart with refreshed items"]
    click node11 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:262:263"
    node10 -->|"No"| node12["Proceed"]
    node11 --> node13{"Is cart obsolete?"}
    node12 --> node13
    click node13 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:266:268"
    node13 -->|"Yes"| node14["Mark cart as obsolete and return"]
    click node14 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:267:269"
    node13 -->|"No"| node15["Return cart"]
    click node15 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:269:269"
    node14 --> node15
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section ensures that the shopping cart accurately reflects the current state of its items, removing obsolete items and marking the cart as obsolete if necessary. It guarantees that only valid items remain in the cart and that the cart's status is updated for further processing.

| Category       | Rule Name                          | Description                                                                                                                                   |
| -------------- | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Empty cart obsolescence            | If the shopping cart is empty or contains no items, the cart must be marked as obsolete and returned immediately.                             |
| Business logic | Item validation and refresh        | Each item in the cart must be validated and updated to reflect its current status, including whether it is obsolete.                          |
| Business logic | Cart obsolescence after item check | If all items in the cart are obsolete after validation, the cart must be marked as obsolete before returning.                                 |
| Business logic | Cart refresh on item removal       | If any items are removed from the cart due to obsolescence, the cart's line items must be refreshed and the cart flagged for database update. |
| Business logic | Return latest cart state           | The returned cart must always reflect the latest valid state, with obsolete items removed and the obsolete flag set if applicable.            |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="225">

---

In `populateShoppingCart`, we first check if the cart or its items are empty—if so, we mark the cart obsolete and bail out. Otherwise, we loop through each item, update it, and check if it's obsolete. If all items are obsolete, the cart gets marked obsolete at the end too.

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

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="250">

---

After updating and checking each item, we build a new set of non-obsolete items. If any items were removed, we refresh the cart's line items and flag that a DB update is needed.

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

Finally, if any items were removed, we update the cart in the DB. If the cart is now obsolete (all items gone or obsolete), we flag it as such before returning it. The returned cart reflects the latest state for the next step.

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

## Post-Processing the Cart After Validation

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Populate shopping cart with current data"]
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:73:73"
    node1 --> node2{"Is shopping cart obsolete?"}
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:74:74"
    node2 -->|"Yes"| node3["Delete shopping cart"]
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:75:75"
    node3 --> node4["Return null"]
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:76:76"
    node2 -->|"No"| node5["Return shopping cart"]
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:78:78"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="73">

---

Back in `getShoppingCart`, after getting the cart from `getByCustomer`, we call `populateShoppingCart` to make sure the cart's items are current and valid before making any decisions based on its state.

```java
			populateShoppingCart(shoppingCart);
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="74">

---

If the cart is obsolete, we delete it and return null; otherwise, we return the cart.

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
