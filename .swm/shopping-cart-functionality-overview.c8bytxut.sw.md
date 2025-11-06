---
title: Shopping Cart Functionality Overview
---
# Shopping Cart Overview

The shopping cart functionality enables customers to add products they wish to purchase, view the quantity of items, see the total price, and remove items from the cart. This core feature facilitates the shopping experience by managing selected products before checkout.

# User Interaction with the Cart

Customers interact with the shopping cart primarily through a mini cart interface accessible in the public shopping section, usually located in the upper menu. This mini cart dynamically displays the number of items and the total price, allowing users to manage their selections seamlessly.

# Frontend Implementation

The add-to-cart functionality is implemented using JavaScript and AJAX, which allows items to be added to the cart without requiring a page reload. This approach enhances user experience by providing immediate feedback and maintaining the shopping flow uninterrupted.

# Backend Cart Management

On the backend, the shopping cart data is managed through a facade layer that abstracts the underlying service operations such as retrieving, updating, and deleting cart contents. This facade interacts with the shopping cart service to maintain the cart's state consistently.

# Session and State Handling

Each shopping cart is associated with a specific customer and merchant store. The cart's state is preserved within the user's session, ensuring that the shopping activity persists across multiple requests and page navigations during the visit.

# Controller Endpoints for Cart Operations

Several REST endpoints in the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/ShoppingCartController.java" pos="91:4:4" line-data="public class ShoppingCartController extends AbstractController {">`ShoppingCartController`</SwmToken> and <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/MiniCartController.java" pos="32:4:4" line-data="public class MiniCartController extends AbstractController{">`MiniCartController`</SwmToken> handle cart operations. These include adding items, removing items, updating quantities, and displaying the cart or mini cart. Each endpoint interacts with the shopping cart facade to perform the necessary business logic and updates the session state accordingly.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/ShoppingCartController.java" line="129">

---

The endpoint `/shop/cart/addShoppingCartItem.html` accepts POST requests with shopping cart item data. It checks for an existing cart in the session or database, creates a new cart if none exists, adds the item, and returns the updated cart data.

```java
    @RequestMapping(value={"/addShoppingCartItem.html"}, method=RequestMethod.POST)
	public @ResponseBody
	ShoppingCartData addShoppingCartItem(@RequestBody final ShoppingCartItem item, final HttpServletRequest request, final HttpServletResponse response, final Locale locale) throws Exception {
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/ShoppingCartController.java" line="307">

---

The endpoint `/shop/cart/removeShoppingCartItem.html` supports GET and POST requests to remove a specific item identified by its line item ID. It updates the cart via the facade and manages the session state to reflect the removal.

```java
	@RequestMapping(value={"/removeShoppingCartItem.html"},   method = { RequestMethod.GET, RequestMethod.POST })

	String removeShoppingCartItem(final Long lineItemId, final HttpServletRequest request, final HttpServletResponse response) throws Exception {
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/ShoppingCartController.java" line="228">

---

The endpoint `/shop/cart/shoppingCart.html` handles GET requests to display the current shopping cart. It retrieves the cart from the session or database and prepares the data for rendering in the user interface.

```java
    @RequestMapping( value = { "/shoppingCart.html" }, method = RequestMethod.GET )
    public String displayShoppingCart( final Model model, final HttpServletRequest request, final HttpServletResponse response, final Locale locale )
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/ShoppingCartController.java" line="367">

---

The endpoint `/shop/cart/updateShoppingCartItem.html` accepts POST requests with an array of shopping cart items to update their quantities. It processes these updates through the facade and returns the updated cart state.

```java
	@RequestMapping(value={"/updateShoppingCartItem.html"},  method = { RequestMethod.POST })
	public @ResponseBody String updateShoppingCartItem( @RequestBody final ShoppingCartItem[] shoppingCartItems, final HttpServletRequest request, final  HttpServletResponse response)  {
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/MiniCartController.java" line="42">

---

The mini cart is managed via endpoints in <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/shoppingCart/MiniCartController.java" pos="32:4:4" line-data="public class MiniCartController extends AbstractController{">`MiniCartController`</SwmToken>. The `/shop/cart/displayMiniCartByCode.html` endpoint retrieves the mini cart based on a cart code, while `/shop/cart/removeMiniShoppingCartItem.html` allows removal of items from the mini cart. Both endpoints update the session state accordingly.

```java
	@RequestMapping(value={"/displayMiniCartByCode.html"},  method = { RequestMethod.GET, RequestMethod.POST })
	public @ResponseBody ShoppingCartData displayMiniCart(final String shoppingCartCode, HttpServletRequest request, Model model){
```

---

</SwmSnippet>

Together, these components provide a robust and user-friendly shopping cart experience, integrating frontend responsiveness with backend reliability and session persistence.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
