---
title: Shopping Cart Architecture and Functionality
---
# Shopping Cart Overview

The shopping cart component in the project manages the products selected by users before they proceed to purchase. It enables users to add items, view quantities and total prices, and remove items from their cart, providing a seamless shopping experience by integrating frontend and backend functionalities.

# Frontend Shopping Cart Interaction

Users interact with the shopping cart primarily through a mini cart accessible from the upper menu in the public shopping section. The add-to-cart functionality is implemented using JavaScript and AJAX, allowing users to add products dynamically without page reloads, which enhances responsiveness and user experience.

# Backend Shopping Cart Architecture

The backend employs a facade pattern to abstract shopping cart operations such as retrieving, updating, and deleting carts. This design ensures a clean separation of concerns and simplifies cart management. Shopping cart data is stored in the user's session and identified by a unique code to maintain state across multiple requests.

# Controller Responsibilities

The `ShoppingCartController` handles user interactions with the cart, including adding and removing items. The `MiniCartController` manages session attributes related to the cart, ensuring the cart's state is preserved throughout the user's session. Business logic for cart operations is encapsulated in `ShoppingCartFacadeImpl`, which the controllers invoke to perform cart manipulations.

# Shopping Cart RESTful Endpoints

The shopping cart exposes RESTful endpoints to manage cart operations. For example, the POST endpoint `/shop/cart/addShoppingCartItem.html` accepts a JSON payload representing an item to add to the cart. This endpoint updates the cart stored in the session or database depending on the user's login state and returns the updated cart data as JSON. It is designed for asynchronous AJAX calls to enable dynamic cart updates without page reloads.

Similarly, the `/shop/cart/removeShoppingCartItem.html` endpoint supports removing items by their line item ID via GET or POST methods, updating the cart accordingly and returning the updated data. The `MiniCartController` provides a related endpoint `/shop/cart/removeMiniShoppingCartItem.html` to handle removals from the mini cart view and update the session state.

Additional endpoints include `/shop/cart/shoppingCart.html` for displaying the full cart page, `/shop/cart/shoppingCartByCode.html` for retrieving a cart by its unique code, and `/shop/cart/updateShoppingCartItem.html` for updating item quantities via POST requests with JSON payloads. Together, these endpoints support the full lifecycle of shopping cart management from the frontend.

# Shopping Cart Facade and Business Logic

The `ShoppingCartFacade` interface and its implementation `ShoppingCartFacadeImpl` encapsulate the business logic for cart operations. They provide methods to add, remove, and update items, retrieve cart data, and delete carts. The facade interacts with services such as `ShoppingCartService` and `ShoppingCartCalculationService` to manage the cart model and calculate totals. It also prepares data transfer objects for controllers to return as responses, promoting a clean separation between business logic and presentation.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
