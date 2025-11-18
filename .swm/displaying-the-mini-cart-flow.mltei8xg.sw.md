---
title: Displaying the Mini Cart Flow
---
This document describes the process of displaying the mini cart by retrieving the current store and customer information, fetching the relevant shopping cart data, handling obsolete carts, and updating the session to ensure the mini cart reflects the correct and personalized shopping cart.

```mermaid
flowchart TD
  node1["Starting the Mini Cart Display Process"]:::HeadingStyle
  node2["Fetching and Validating the Shopping Cart
Retrieve cart for customer or fallback by cart code
(Fetching and Validating the Shopping Cart)"]:::HeadingStyle
  node3["Fetching and Validating the Shopping Cart
Is cart found and not obsolete?
(Fetching and Validating the Shopping Cart)"]:::HeadingStyle
  node4["Fetching and Validating the Shopping Cart
Delete obsolete cart and return null
(Fetching and Validating the Shopping Cart)"]:::HeadingStyle
  node5["Fetching and Validating the Shopping Cart
Populate cart with pricing and language
(Fetching and Validating the Shopping Cart)"]:::HeadingStyle
  node6["Updating Session and Returning Cart Data"]:::HeadingStyle

  node1 --> node2
  node2 --> node3
  node3 -- No --> node4
  node3 -- Yes --> node5
  node5 --> node6

  click node1 goToHeading "Starting the Mini Cart Display Process"
  click node2 goToHeading "Fetching and Validating the Shopping Cart"
  click node3 goToHeading "Fetching and Validating the Shopping Cart"
  click node4 goToHeading "Fetching and Validating the Shopping Cart"
  click node5 goToHeading "Fetching and Validating the Shopping Cart"
  click node6 goToHeading "Updating Session and Returning Cart Data"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Starting the Mini Cart Display Process

This section handles the process of displaying the mini cart by retrieving the store and customer information and then requesting the cart data from the facade, keeping the controller clean.

| Category       | Rule Name                  | Description                                                                                                                               |
| -------------- | -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Store Context Retrieval    | The mini cart display must always retrieve the current store context to ensure the cart data is relevant to the correct merchant store.   |
| Business logic | Customer Session Retrieval | The mini cart display must retrieve the current customer session to personalize the cart data for the logged-in user or guest.            |
| Business logic | Cart Data Fetching         | The mini cart data must be fetched using the shopping cart code, customer, and store information to ensure the correct cart is displayed. |

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/MiniCartController.java" line="43">

---

We get the store and customer info, then ask the facade for the cart data, keeping the controller clean.

```java
	public @ResponseBody ShoppingCartData displayMiniCart(final String shoppingCartCode, HttpServletRequest request, Model model){
		
		try {
			MerchantStore merchantStore = (MerchantStore)request.getAttribute(Constants.MERCHANT_STORE);
		    Customer customer = getSessionAttribute(  Constants.CUSTOMER, request );
			ShoppingCartData cart =  shoppingCartFacade.getShoppingCartData(customer,merchantStore,shoppingCartCode);
```

---

</SwmSnippet>

## Fetching and Validating the Shopping Cart

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is customer provided?"}
    click node1 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java:268:270"
    node1 -->|"Yes"| node2["Retrieve cart for customer"]
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java:68:79"
    node1 -->|"No"| node3{"Is shopping cart ID provided and cart not found?"}
    click node3 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java:278:281"
    node3 -->|"Yes"| node4["Retrieve cart by shopping cart ID"]
    click node4 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java:278:281"
    node3 -->|"No"| node5["No cart found, return null"]
    click node5 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java:293:295"
    node2 --> node6{"Is cart found?"}
    click node6 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java:293:295"
    node4 --> node6
    node6 -->|"No"| node5
    node6 -->|"Yes"| node7["Populate shopping cart data with pricing and language"]
    click node7 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java:300:306"
    node7 --> node8["Return populated shopping cart data"]
    click node8 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java:306:307"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1{"Is customer provided?"}
%%     click node1 openCode "<SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>:268:270"
%%     node1 -->|"Yes"| node2["Retrieve cart for customer"]
%%     click node2 openCode "<SwmPath>[shopizer/…/service/ShoppingCartServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java)</SwmPath>:68:79"
%%     node1 -->|"No"| node3{"Is shopping cart ID provided and cart not found?"}
%%     click node3 openCode "<SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>:278:281"
%%     node3 -->|"Yes"| node4["Retrieve cart by shopping cart ID"]
%%     click node4 openCode "<SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>:278:281"
%%     node3 -->|"No"| node5["No cart found, return null"]
%%     click node5 openCode "<SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>:293:295"
%%     node2 --> node6{"Is cart found?"}
%%     click node6 openCode "<SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>:293:295"
%%     node4 --> node6
%%     node6 -->|"No"| node5
%%     node6 -->|"Yes"| node7["Populate shopping cart data with pricing and language"]
%%     click node7 openCode "<SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>:300:306"
%%     node7 --> node8["Return populated shopping cart data"]
%%     click node8 openCode "<SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>:306:307"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section handles fetching and validating the shopping cart for a customer or by shopping cart ID, ensuring the cart is current and populated with necessary pricing and language data before returning it.

| Category       | Rule Name                               | Description                                                                                                                        |
| -------------- | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Customer cart retrieval                 | If a customer is provided, retrieve the shopping cart associated with that customer.                                               |
| Business logic | Fallback cart retrieval by ID           | If no customer is provided or no cart is found for the customer, and a shopping cart ID is provided, retrieve the cart by that ID. |
| Business logic | Obsolete cart handling                  | If a retrieved shopping cart is marked as obsolete, delete it and return null instead of the cart.                                 |
| Business logic | Return null if no cart found            | If no shopping cart is found by customer or ID, return null to indicate absence of a valid cart.                                   |
| Business logic | Populate cart with pricing and language | Before returning a valid cart, populate it with pricing calculations and language-specific data.                                   |

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="260">

---

In this snippet, we check if there's a customer and then call the service to get their cart. This delegation lets the service handle the details of cart retrieval, keeping the facade focused on flow control.

```java
    public ShoppingCartData getShoppingCartData( final Customer customer, final MerchantStore store,
                                                 final String shoppingCartId )
        throws Exception
    {

        ShoppingCart cart = null;
        try
        {
            if ( customer != null )
            {
                LOG.info( "Reteriving customer shopping cart..." );

                cart = shoppingCartService.getShoppingCart( customer );

            }

            else
            {
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/shoppingcart/service/ShoppingCartServiceImpl.java" line="68">

---

Here we retrieve the cart for a customer, fill it with necessary details, then check if it's obsolete. If it is, we delete it and return null; otherwise, we return the cart.

```java
	public ShoppingCart getShoppingCart(final Customer customer) throws ServiceException {

		try {

			ShoppingCart shoppingCart = shoppingCartDao.getByCustomer(customer);
			populateShoppingCart(shoppingCart);
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

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="278">

---

After getting the cart from the service, if it's null and we have a cart code, we try to get the cart by that code. Then we populate the cart data with pricing and language info before returning it.

```java
                if ( StringUtils.isNotBlank( shoppingCartId ) && cart == null )
                {
                    cart = shoppingCartService.getByCode( shoppingCartId, store );
                }

            }
        }
        catch ( ServiceException ex )
        {
            LOG.error( "Error while retriving cart from customer", ex );
        }
        catch( NoResultException nre) {
        	//nothing
        }

        if ( cart == null )
        {
            return null;
        }

        LOG.info( "Cart model found." );

        ShoppingCartDataPopulator shoppingCartDataPopulator = new ShoppingCartDataPopulator();
        shoppingCartDataPopulator.setShoppingCartCalculationService( shoppingCartCalculationService );
        shoppingCartDataPopulator.setPricingService( pricingService );

        Language language = (Language) getKeyValue( Constants.LANGUAGE );
        MerchantStore merchantStore = (MerchantStore) getKeyValue( Constants.MERCHANT_STORE );
        return shoppingCartDataPopulator.populate( cart, merchantStore, language );

    }
```

---

</SwmSnippet>

## Updating Session and Returning Cart Data

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/MiniCartController.java" line="49">

---

After getting the cart data, we update the session with the cart code if present, or remove the cart attribute if not. Then we return the cart data to the caller.

```java
			if(cart!=null) {
				request.getSession().setAttribute(Constants.SHOPPING_CART, cart.getCode());
			}
			if(cart==null) {
				request.getSession().removeAttribute(Constants.SHOPPING_CART);//make sure there is no cart here
			}
			return cart;
			
			
		} catch(Exception e) {
			LOG.error("Error while getting the shopping cart",e);
		}
		
		return null;

	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
