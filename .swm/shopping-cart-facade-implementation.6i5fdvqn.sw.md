---
title: Shopping Cart Facade Implementation
---
# Introduction

This document explains the main ideas behind the implementation of the shopping cart facade in <SwmPath>[shopizer/…/facade/ShoppingCartFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java)</SwmPath>. The facade acts as the main entry point for shopping cart operations from the web layer, abstracting the underlying service logic and data model.

We will cover:

1. Why the facade pattern is used for shopping cart operations.
2. How adding items to the cart is handled, including duplicate detection and attribute management.
3. The approach for updating and removing cart items.
4. How cart data is retrieved and mapped for the web layer.
5. The rationale for cart creation and cart model retrieval.

# Why use a facade for shopping cart operations

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="45">

---

The facade pattern is used here to centralize and simplify shopping cart operations for the web layer. This class is annotated as a Spring <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" pos="50:0:1" line-data="@Service( value = &quot;shoppingCartFacade&quot; )">`@Service`</SwmToken> and implements the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" pos="52:3:3" line-data="    implements ShoppingCartFacade">`ShoppingCartFacade`</SwmToken> interface, making it injectable and testable. All dependencies on lower-level services (cart, product, pricing, etc.) are injected, so the controller layer doesn't need to know about the details.

```java
/**
 * @author Umesh Awasthi
 * @version 1.2
 * @since 1.2
 */
@Service( value = "shoppingCartFacade" )
public class ShoppingCartFacadeImpl
    implements ShoppingCartFacade
{
```

---

</SwmSnippet>

# Adding items to the cart: duplicate detection and attribute handling

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="97">

---

When adding an item to the cart, the code first tries to retrieve the existing cart by code. If it doesn't exist, it creates a new cart. This ensures that cart operations are always performed on a valid cart model.

```java
        ShoppingCart cartModel = null;
        if ( !StringUtils.isBlank( item.getCode() ) )
        {
            // get it from the db
            cartModel = getShoppingCartModel( item.getCode(), store );
            if ( cartModel == null )
            {
                cartModel = createCartModel( shoppingCartData.getCode(), store,customer );
            }

        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="109">

---

If the cart is still null, a new cart is created, either with a provided code or a generated one. The item is then created and prepared for addition.

```java
        if ( cartModel == null )
        {

            final String shoppingCartCode =
                StringUtils.isNotBlank( shoppingCartData.getCode() ) ? shoppingCartData.getCode() : null;
            cartModel = createCartModel( shoppingCartCode, store,customer );

        }
        com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem shoppingCartItem =
            createCartItem( cartModel, item, store );
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="120">

---

The code checks for duplicates by comparing product IDs and attributes. If a duplicate is found (same product, no attributes), it increments the quantity instead of adding a new line item. This avoids cluttering the cart with multiple identical entries.

```java
        boolean duplicateFound = false;
        if(CollectionUtils.isEmpty(item.getShoppingCartAttributes())) {//increment quantity
        	//get duplicate item from the cart
        	Set<com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem> cartModelItems = cartModel.getLineItems();
        	for(com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem cartItem : cartModelItems) {
        		if(cartItem.getProduct().getId().longValue()==shoppingCartItem.getProduct().getId().longValue()) {
        			if(CollectionUtils.isEmpty(cartItem.getAttributes())) {
        				if(!duplicateFound) {
        					if(!shoppingCartItem.isProductVirtual()) {
	        					cartItem.setQuantity(cartItem.getQuantity() + shoppingCartItem.getQuantity());
        					}
        					duplicateFound = true;
        					break;
        				}
        			}
        		}
        	}
        } 
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="139">

---

If no duplicate is found, the new item is added to the cart, and the cart is persisted.

```java
        if(!duplicateFound) {
        	cartModel.getLineItems().add( shoppingCartItem );
        }
        
        /** Update cart in database with line items **/
        shoppingCartService.saveOrUpdate( cartModel );
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="146">

---

After updating, the cart is refreshed and recalculated to ensure totals and prices are up to date.

```java
        //refresh cart
        cartModel = shoppingCartService.getById(cartModel.getId(), store);

        shoppingCartCalculationService.calculate( cartModel, store, language );
```

---

</SwmSnippet>

# Cart item creation and attribute mapping

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="158">

---

When creating a cart item, the code validates that the product exists and belongs to the correct merchant. It then populates the cart item, sets the quantity, and attaches any product attributes (such as size or color) that the user selected.

```java
    private com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem createCartItem( final ShoppingCart cartModel,
                                                                                               final ShoppingCartItem shoppingCartItem,
                                                                                               final MerchantStore store )
        throws Exception
    {
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="164">

---

The product is fetched and validated for existence.

```java
        Product product = productService.getById( shoppingCartItem.getProductId() );

        if ( product == null )
        {
            throw new Exception( "Item with id " + shoppingCartItem.getProductId() + " does not exist" );
        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="171">

---

It is also checked that the product belongs to the current merchant store, preventing cross-store contamination.

```java
        if ( product.getMerchantStore().getId().intValue() != store.getId().intValue() )
        {
            throw new Exception( "Item with id " + shoppingCartItem.getProductId() + " does not belong to merchant "
                + store.getId() );
        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="177">

---

The cart item is populated and linked to the cart.

```java
        com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem item =
            shoppingCartService.populateShoppingCartItem( product );

        item.setQuantity( shoppingCartItem.getQuantity() );
        item.setShoppingCart( cartModel );
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="183">

---

Attributes are mapped by fetching the corresponding product attributes and associating them with the cart item, but only if they belong to the same product.

```java
        // attributes
        List<ShoppingCartAttribute> cartAttributes = shoppingCartItem.getShoppingCartAttributes();
        if ( !CollectionUtils.isEmpty( cartAttributes ) )
        {
            for ( ShoppingCartAttribute attribute : cartAttributes )
            {
                ProductAttribute productAttribute = productAttributeService.getById( attribute.getAttributeId() );
                if ( productAttribute != null
                    && productAttribute.getProduct().getId().longValue() == product.getId().longValue() )
                {
                    com.salesmanager.core.business.shoppingcart.model.ShoppingCartAttributeItem attributeItem =
                        new com.salesmanager.core.business.shoppingcart.model.ShoppingCartAttributeItem( item,
                                                                                                         productAttribute );
```

---

</SwmSnippet>

# Removing and updating cart items

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="330">

---

Removing an item from the cart is handled by filtering out the item with the given ID and saving the updated cart. This approach is simple and avoids mutating the set while iterating.

```java
            ShoppingCart cartModel = getCartModel( cartId,store );
            if ( cartModel != null )
            {
                if ( CollectionUtils.isNotEmpty( cartModel.getLineItems() ) )
                {
                    Set<com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem> shoppingCartItemSet =
                        new HashSet<com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem>();
                    for ( com.salesmanager.core.business.shoppingcart.model.ShoppingCartItem shoppingCartItem : cartModel.getLineItems() )
                    {
                        if ( shoppingCartItem.getId().longValue() != itemID.longValue() )
                        {
                            shoppingCartItemSet.add( shoppingCartItem );
                        }
                    }
                    cartModel.setLineItems( shoppingCartItemSet );
                    shoppingCartService.saveOrUpdate( cartModel );
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="350">

---

After removal, the updated cart is mapped back to a DTO for the web layer.

```java
                    ShoppingCartDataPopulator shoppingCartDataPopulator = new ShoppingCartDataPopulator();
                    shoppingCartDataPopulator.setShoppingCartCalculationService( shoppingCartCalculationService );
                    shoppingCartDataPopulator.setPricingService( pricingService );
                    return shoppingCartDataPopulator.populate( cartModel, store, language );
                }
            }
        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="381">

---

Updating a cart item checks that the new quantity is valid, finds the entry to update, and recalculates the price based on the product's attributes. This ensures that price changes (e.g., due to attribute selection) are reflected immediately.

```java
                entryToUpdate.getProduct();

                LOG.info( "Updating cart entry quantity to" + newQuantity );
                entryToUpdate.setQuantity( (int) newQuantity );
                List<ProductAttribute> productAttributes = new ArrayList<ProductAttribute>();
                productAttributes.addAll( entryToUpdate.getProduct().getAttributes() );
                final FinalPrice finalPrice =
                    productPriceUtils.getFinalProductPrice( entryToUpdate.getProduct(), productAttributes );
                entryToUpdate.setItemPrice( finalPrice.getFinalPrice() );
                shoppingCartService.saveOrUpdate( cartModel );
```

---

</SwmSnippet>

# Retrieving and mapping cart data

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="265">

---

Retrieving cart data is done either by customer or by cart code. The code tries to fetch the cart for the customer first, falling back to the cart code if necessary. This supports both logged-in and guest users.

```java
        ShoppingCart cart = null;
        try
        {
            if ( customer != null )
            {
                LOG.info( "Reteriving customer shopping cart..." );
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="276">

---

If the customer is not present, the cart is fetched by code.

```java
            else
            {
                if ( StringUtils.isNotBlank( shoppingCartId ) && cart == null )
                {
                    cart = shoppingCartService.getByCode( shoppingCartId, store );
                }

            }
        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="298">

---

The cart is then mapped to a DTO using a populator, which attaches pricing and calculation services. This keeps the web layer decoupled from the internal cart model.

```java
        LOG.info( "Cart model found." );

        ShoppingCartDataPopulator shoppingCartDataPopulator = new ShoppingCartDataPopulator();
        shoppingCartDataPopulator.setShoppingCartCalculationService( shoppingCartCalculationService );
        shoppingCartDataPopulator.setPricingService( pricingService );
```

---

</SwmSnippet>

# Cart creation and model retrieval

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="205">

---

Cart creation generates a new cart with a unique code if none is provided, associates it with the merchant and customer, and persists it. This ensures every cart is uniquely identifiable and linked to the correct context.

```java
    @Override
    public ShoppingCart createCartModel( final String shoppingCartCode, final MerchantStore store,final Customer customer )
        throws Exception
    {
        final Long CustomerId = customer != null ? customer.getId() : null;
        ShoppingCart cartModel = new ShoppingCart();
        if ( StringUtils.isNotBlank( shoppingCartCode ) )
        {
            cartModel.setShoppingCartCode( shoppingCartCode );
        }
        else
        {
            cartModel.setShoppingCartCode( UUID.randomUUID().toString().replaceAll( "-", "" ) );
        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="220">

---

After setting up the cart, it is saved and returned for further operations.

```java
        cartModel.setMerchantStore( store );
        if ( CustomerId != null )
        {
            cartModel.setCustomerId( CustomerId );
            ;
        }
        shoppingCartService.create( cartModel );
        return cartModel;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/facade/ShoppingCartFacadeImpl.java" line="459">

---

Retrieving a cart model by code is wrapped in error handling to avoid propagating exceptions to the web layer, returning null if not found.

```java
    private ShoppingCart getCartModel( final String cartId,final MerchantStore store )
    {
        if ( StringUtils.isNotBlank( cartId ) )
        {
           try
            {
                return shoppingCartService.getByCode( cartId, store );
            }
            catch ( ServiceException e )
            {
                LOG.error( "unable to find any cart asscoiated with this Id: " + cartId );
                LOG.error( "error while fetching cart model...", e );
                return null;
            }
            catch( NoResultException nre) {
           	//nothing
            }

        }
        return null;
    }
```

---

</SwmSnippet>

This approach keeps the facade robust and user-facing operations predictable.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
