---
title: Displaying Shopping Cart Items
---
This document describes how shopping cart items are presented to users. The flow receives shopping cart data and a target UI element, prepares the data for display, and updates the interface to show either an empty cart message or the current cart items.

# Where is this flow used?

This flow is used multiple times in the codebase as represented in the following diagram:

```mermaid
graph TD;
      1f5828ad4614d62214c21a93adad27e2043215eeadb8878acef5e01eaa2c8439(shopping-cart.js::success) --> 58dea14ac261f29155c2e7c602910cc5e8a74ffa23df79f1b0828a3a56bc87a5(shopping-cart.js::displayShoppigCartItems):::mainFlowStyle

68069365ebe618b22ec81e10de3eafbcd7909f7c4a2932ff1e995fdca622cab3(shopping-cart.js::displayMiniCart) --> 58dea14ac261f29155c2e7c602910cc5e8a74ffa23df79f1b0828a3a56bc87a5(shopping-cart.js::displayShoppigCartItems):::mainFlowStyle

69f39dcea00daab92763cd1592260d45f5f6d1ba87f3dbb953bbb91c16f1f5fd(shopping-cart.js::initBindings) --> 68069365ebe618b22ec81e10de3eafbcd7909f7c4a2932ff1e995fdca622cab3(shopping-cart.js::displayMiniCart)

69f39dcea00daab92763cd1592260d45f5f6d1ba87f3dbb953bbb91c16f1f5fd(shopping-cart.js::initBindings) --> 9f21ac5b0affeff95f82edd873db6b4c6bc42faa456a4b91f904f46e655d30ef(shopping-cart.js::addToCart)

1f5828ad4614d62214c21a93adad27e2043215eeadb8878acef5e01eaa2c8439(shopping-cart.js::success) --> 58dea14ac261f29155c2e7c602910cc5e8a74ffa23df79f1b0828a3a56bc87a5(shopping-cart.js::displayShoppigCartItems):::mainFlowStyle

fe6237905da88029dec7d8d46bb8eadd35fc2021ad79899e63ecc106b8571c07(shopping-cart.js::removeItemFromMinicart) --> 58dea14ac261f29155c2e7c602910cc5e8a74ffa23df79f1b0828a3a56bc87a5(shopping-cart.js::displayShoppigCartItems):::mainFlowStyle

1f5828ad4614d62214c21a93adad27e2043215eeadb8878acef5e01eaa2c8439(shopping-cart.js::success) --> 58dea14ac261f29155c2e7c602910cc5e8a74ffa23df79f1b0828a3a56bc87a5(shopping-cart.js::displayShoppigCartItems):::mainFlowStyle

9f21ac5b0affeff95f82edd873db6b4c6bc42faa456a4b91f904f46e655d30ef(shopping-cart.js::addToCart) --> 58dea14ac261f29155c2e7c602910cc5e8a74ffa23df79f1b0828a3a56bc87a5(shopping-cart.js::displayShoppigCartItems):::mainFlowStyle


classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9
```

# Preparing Cart Data and Template

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" line="331">

---

We prep the cart data and template, and clear the div so it's ready for new cart markup. The next step is to use jQuery's html function to do the clearing.

```javascript
function displayShoppigCartItems(cart, div) {
	
	 
	//set cart contextPath
	cart.contextPath=getContextPath(); 
	var template = Hogan.compile(document.getElementById("miniShoppingCartTemplate").innerHTML);
	
    
	 $(div).html('');
```

---

</SwmSnippet>

## Efficiently Updating Cart DOM

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="5829">

---

In `html`, we check if we're getting or setting HTML. If setting, and the value is a simple string, we use innerHTML directly for speed. This is why we call this after prepping the cart template, to quickly clear the div before rendering new content.

```javascript
	html: function( value ) {
		if ( value === undefined ) {
			return this[0] && this[0].nodeType === 1 ?
				this[0].innerHTML.replace(rinlinejQuery, "") :
				null;

		// See if we can take a shortcut and just use innerHTML
		} else if ( typeof value === "string" && !rnoInnerhtml.test( value ) &&
			(jQuery.support.leadingWhitespace || !rleadingWhitespace.test( value )) &&
			!wrapMap[ (rtagName.exec( value ) || ["", ""])[1].toLowerCase() ] ) {

			value = value.replace(rxhtmlTag, "<$1></$2>");

			try {
				for ( var i = 0, l = this.length; i < l; i++ ) {
					// Remove element nodes and prevent memory leaks
					if ( this[i].nodeType === 1 ) {
						jQuery.cleanData( this[i].getElementsByTagName("*") );
						this[i].innerHTML = value;
					}
				}

```

---

</SwmSnippet>

### Cleaning Up Old jQuery Data

See <SwmLink doc-title="Cleaning Up DOM Elements">[Cleaning Up DOM Elements](\.swm\cleaning-up-dom-elements.qc62kj2s.sw.md)</SwmLink>

### Fallbacks and Dynamic HTML Updates

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is value a function?"}
    click node1 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:5856:5865"
    node1 -->|"Yes"| node2["Generate and set HTML for each element"]
    click node2 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:5857:5861"
    node1 -->|"No"| node3["Set HTML content for all elements"]
    click node3 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:5853:5854"
    node2 --> node4["Return elements"]
    click node4 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:5867:5868"
    node3 --> node4

    subgraph loop1["For each element"]
        node2 --> node5["Call function to generate HTML"]
        click node5 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:5860:5861"
        node5 --> node2
    end

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="5851">

---

Back in the `html` function, if innerHTML throws, we fall back to emptying and appending the new content. If the value is a function, we call it for each element to set dynamic HTML. Otherwise, we just empty and append. This keeps the cart update robust after clearing the div.

```javascript
			// If using innerHTML throws an exception, use the fallback method
			} catch(e) {
				this.empty().append( value );
			}

		} else if ( jQuery.isFunction( value ) ) {
			this.each(function(i){
				var self = jQuery( this );

				self.html( value.call(this, i, self.html()) );
			});

		} else {
			this.empty().append( value );
		}

		return this;
	},
```

---

</SwmSnippet>

## Rendering and Displaying Cart Items

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start: Display shopping cart items"]
    click node1 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:340:352"
    node1 --> node2{"Is shoppingCartItems empty?"}
    click node2 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:340:343"
    node2 -->|"Yes"| node3["Show empty cart message and exit"]
    click node3 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:341:342"
    node2 -->|"No"| node4["Hide cart message"]
    click node4 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:345:345"
    node4 --> node5["Show shopping cart UI"]
    click node5 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:346:346"
    node5 --> node6["Render cart items with template"]
    click node6 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:349:349"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" line="340">

---

After returning from the jQuery html function, in `displayShoppigCartItems` we check if the cart is empty and handle that case. Otherwise, we hide the cart message, show the cart, and render the cart items using the Hogan template. This updates the UI with the latest cart data.

```javascript
	 if(cart.shoppingCartItems==null) {
		 emptyCartLabel();
		 return;
	 }
	 
	 $('#cartMessage').hide();
	 $('#shoppingcart').show();

	 //call template defined in template directory
	 $(div).append(template.render(cart));


}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
