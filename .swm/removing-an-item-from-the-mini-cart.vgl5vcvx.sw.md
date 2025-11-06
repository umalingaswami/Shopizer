---
title: Removing an item from the mini cart
---
This document describes how users can remove items from their mini cart and how the cart display is updated. When a user requests to remove an item, the system updates the cart to show either the remaining items and totals or an empty cart label.

# Removing an item from the minicart

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["User requests to remove item (lineItemId) from mini cart"]
  click node1 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:275:299"
  node1 --> node2["Send removal request with shoppingCartCode"]
  click node2 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:278:298"
  node2 --> node3{"Is mini cart empty or has no items?"}
  click node3 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:287:295"
  node3 -->|"Yes"| node4["Show empty cart label"]
  click node4 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:288:294"
  node3 -->|"No"| node5["Display remaining cart items and totals"]
  click node5 openCode "shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js:291:292"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["User requests to remove item (<SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="275:4:4" line-data="function removeItemFromMinicart(lineItemId){">`lineItemId`</SwmToken>) from mini cart"]
%%   click node1 openCode "<SwmPath>[shopizer/…/js/shopping-cart.js](shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js)</SwmPath>:275:299"
%%   node1 --> node2["Send removal request with <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="277:1:1" line-data="	shoppingCartCode = getCartCode();">`shoppingCartCode`</SwmToken>"]
%%   click node2 openCode "<SwmPath>[shopizer/…/js/shopping-cart.js](shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js)</SwmPath>:278:298"
%%   node2 --> node3{"Is mini cart empty or has no items?"}
%%   click node3 openCode "<SwmPath>[shopizer/…/js/shopping-cart.js](shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js)</SwmPath>:287:295"
%%   node3 -->|"Yes"| node4["Show empty cart label"]
%%   click node4 openCode "<SwmPath>[shopizer/…/js/shopping-cart.js](shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js)</SwmPath>:288:294"
%%   node3 -->|"No"| node5["Display remaining cart items and totals"]
%%   click node5 openCode "<SwmPath>[shopizer/…/js/shopping-cart.js](shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js)</SwmPath>:291:292"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:<SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="1470:10:10" line-data="		div.style.width = &quot;2px&quot;;">`2px`</SwmToken>;
```

This section governs the rules for removing an item from the mini cart, ensuring the correct cart is targeted, the item is removed, and the cart display is updated appropriately for the user.

| Category        | Rule Name                 | Description                                                                                                         |
| --------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Data validation | Store-scoped cart removal | Only items belonging to the cart associated with the current merchant store may be removed from the mini cart.      |
| Business logic  | Empty cart display        | If the mini cart becomes empty after item removal, the user must be shown an empty cart label.                      |
| Business logic  | Updated cart display      | If items remain in the mini cart after removal, the cart must display the updated list of items and the new totals. |

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" line="275">

---

In <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="275:2:2" line-data="function removeItemFromMinicart(lineItemId){">`removeItemFromMinicart`</SwmToken>, we start by grabbing the cart code using <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="277:5:5" line-data="	shoppingCartCode = getCartCode();">`getCartCode`</SwmToken>. This is needed because just having the <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="275:4:4" line-data="function removeItemFromMinicart(lineItemId){">`lineItemId`</SwmToken> isn't enough—the backend needs to know which cart to update, especially if there are multiple stores or carts in play. Next, we call <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="277:5:5" line-data="	shoppingCartCode = getCartCode();">`getCartCode`</SwmToken> to extract that identifier from the user's cookie.

```javascript
function removeItemFromMinicart(lineItemId){
	
	shoppingCartCode = getCartCode();
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" line="367">

---

<SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="367:2:2" line-data="function getCartCode() {">`getCartCode`</SwmToken> pulls the cart cookie, splits it into store code and cart ID, and checks if the store code matches the current merchant store. If it matches, we use the cart ID; otherwise, we bail out with undefined. This keeps carts scoped to the right store.

```javascript
function getCartCode() {
	
	var cart = $.cookie('cart'); //should be [storecode_cartid]
	var code = new Array();
	
	if(cart!=null) {
		code = cart.split('_');
		if(code[0]==getMerchantStoreCode()) {
			return code[1];
		}
	}
}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" line="278">

---

Back in <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/shopping-cart.js" pos="275:2:2" line-data="function removeItemFromMinicart(lineItemId){">`removeItemFromMinicart`</SwmToken>, after getting the cart code, we fire off an AJAX request to remove the item from the server-side cart. The response is used to update the minicart UI—either showing the updated items and totals or clearing the cart label if it's empty. We call into the <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7210:5:5" line-data="			s = jQuery.ajaxSetup( {}, options ),">`jQuery`</SwmToken> AJAX logic next to handle the request and response asynchronously.

```javascript
	$.ajax({  
		 type: 'GET',
		 cache:false,
		 url: getContextPath() + '/shop/cart/removeMiniShoppingCartItem.html?lineItemId='+lineItemId + '&shoppingCartCode=' + shoppingCartCode,  
		 error: function(e) { 
			 console.log('error ' + e);
			 
		 },
		 success: function(miniCart) {
			 if(miniCart==null) {
				 emptyCartLabel();
			 } else {
				 if(miniCart.shoppingCartItems!=null) {
					 displayShoppigCartItems(miniCart,'#shoppingcartProducts');
					 displayTotals(miniCart);
				 } else {
					 emptyCartLabel();
				 }
			 }
		} 
	});
}
```

---

</SwmSnippet>

# Sending the AJAX request and handling the response

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Receive AJAX request input"] --> node2{"Is input a URL or options object?"}
    click node1 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7198:7204"
    node2 -->|"Options object"| node3["Normalize input"]
    click node2 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7201:7204"
    node2 -->|"URL"| node3
    node3 --> node4["Configure request options"]
    click node3 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7206:7210"
    node4 --> node5["Set up headers and event callbacks"]
    click node4 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7527:7543"
    
    subgraph loop1["For each header and event callback"]
        node5 -->|"For each header/callback"| node6["Apply header or callback to request"]
        click node6 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7527:7543"
    end
    node5 --> node7{"Should request be aborted before sending?"}
    click node7 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7533:7536"
    node7 -->|"Yes"| node8["Abort request"]
    click node8 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7288:7295"
    node7 -->|"No"| node9{"Is transport available?"}
    click node9 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7546:7551"
    node9 -->|"No"| node10["Fail request: No transport"]
    click node10 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7550:7551"
    node9 -->|"Yes"| node11["Send request"]
    click node11 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7552:7567"
    node11 --> node12{"Was request successful?"}
    click node12 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7334:7367"
    node12 -->|"Success"| node13["Trigger success events"]
    click node13 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7384:7386"
    node12 -->|"Error"| node14["Trigger error events"]
    click node14 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7387:7388"
    node13 --> node15["Return AJAX response object"]
    click node15 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:7578:7579"
    node14 --> node15
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Receive AJAX request input"] --> node2{"Is input a URL or options object?"}
%%     click node1 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7198:7204"
%%     node2 -->|"Options object"| node3["Normalize input"]
%%     click node2 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7201:7204"
%%     node2 -->|"URL"| node3
%%     node3 --> node4["Configure request options"]
%%     click node3 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7206:7210"
%%     node4 --> node5["Set up headers and event callbacks"]
%%     click node4 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7527:7543"
%%     
%%     subgraph loop1["For each header and event callback"]
%%         node5 -->|"For each header/callback"| node6["Apply header or callback to request"]
%%         click node6 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7527:7543"
%%     end
%%     node5 --> node7{"Should request be aborted before sending?"}
%%     click node7 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7533:7536"
%%     node7 -->|"Yes"| node8["Abort request"]
%%     click node8 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7288:7295"
%%     node7 -->|"No"| node9{"Is transport available?"}
%%     click node9 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7546:7551"
%%     node9 -->|"No"| node10["Fail request: No transport"]
%%     click node10 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7550:7551"
%%     node9 -->|"Yes"| node11["Send request"]
%%     click node11 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7552:7567"
%%     node11 --> node12{"Was request successful?"}
%%     click node12 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7334:7367"
%%     node12 -->|"Success"| node13["Trigger success events"]
%%     click node13 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7384:7386"
%%     node12 -->|"Error"| node14["Trigger error events"]
%%     click node14 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7387:7388"
%%     node13 --> node15["Return AJAX response object"]
%%     click node15 openCode "<SwmPath>[shopizer/…/bootstrap/jquery.js](shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js)</SwmPath>:7578:7579"
%%     node14 --> node15
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:<SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="1470:10:10" line-data="		div.style.width = &quot;2px&quot;;">`2px`</SwmToken>;
```

This section governs how AJAX requests are initiated, configured, sent, and how their responses are handled in Shopizer. It ensures requests are properly normalized, headers and callbacks are set, aborts and errors are managed, and the final response object is returned for further use.

| Category       | Rule Name                | Description                                                                                                                                                                                                                                                                                                                                              |
| -------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Header Application       | All request headers specified in the options must be applied to the outgoing AJAX request.                                                                                                                                                                                                                                                               |
| Business logic | Pre-send Abort           | If the <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7533:7:7" line-data="		if ( s.beforeSend &amp;&amp; ( s.beforeSend.call( callbackContext, jqXHR, s ) === false \|\| state === 2 ) ) {">`beforeSend`</SwmToken> callback returns false, the AJAX request must be aborted and no network call should be made. |
| Business logic | Success Event Triggering | On successful completion of the AJAX request (HTTP status 200-299 or 304), success events and callbacks must be triggered.                                                                                                                                                                                                                               |

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="7198">

---

In <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7198:1:1" line-data="	ajax: function( url, options ) {">`ajax`</SwmToken>, we kick off by normalizing the arguments—either a URL or an options object. Then we set up deferreds and promises for async handling, and build out the request headers and event contexts. This sets up the groundwork for the AJAX lifecycle, including flexible callback management and header setup.

```javascript
	ajax: function( url, options ) {

		// If url is an object, simulate pre-1.5 signature
		if ( typeof url === "object" ) {
			options = url;
			url = undefined;
		}

		// Force options to be an object
		options = options || {};

		var // Create the final options object
			s = jQuery.ajaxSetup( {}, options ),
			// Callbacks context
			callbackContext = s.context || s,
			// Context for global events
			// It's the callbackContext if one was provided in the options
			// and if it's a DOM node or a jQuery collection
			globalEventContext = callbackContext !== s &&
				( callbackContext.nodeType || callbackContext instanceof jQuery ) ?
						jQuery( callbackContext ) : jQuery.event,
			// Deferreds
			deferred = jQuery.Deferred(),
			completeDeferred = jQuery.Callbacks( "once memory" ),
			// Status-dependent callbacks
			statusCode = s.statusCode || {},
			// ifModified key
			ifModifiedKey,
			// Headers (they are sent all at once)
			requestHeaders = {},
			requestHeadersNames = {},
			// Response headers
			responseHeadersString,
			responseHeaders,
			// transport
			transport,
			// timeout handle
			timeoutTimer,
			// Cross-domain detection vars
			parts,
			// The jqXHR state
			state = 0,
			// To know if global events are to be dispatched
			fireGlobals,
			// Loop variable
			i,
			// Fake xhr
			jqXHR = {

				readyState: 0,

				// Caches the header
				setRequestHeader: function( name, value ) {
					if ( !state ) {
						var lname = name.toLowerCase();
						name = requestHeadersNames[ lname ] = requestHeadersNames[ lname ] || name;
						requestHeaders[ name ] = value;
					}
					return this;
				},

				// Raw string
				getAllResponseHeaders: function() {
					return state === 2 ? responseHeadersString : null;
				},

				// Builds headers hashtable if needed
				getResponseHeader: function( key ) {
					var match;
					if ( state === 2 ) {
						if ( !responseHeaders ) {
							responseHeaders = {};
							while( ( match = rheaders.exec( responseHeadersString ) ) ) {
								responseHeaders[ match[1].toLowerCase() ] = match[ 2 ];
							}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="7274">

---

Here we manage request and response headers, including custom ones and conditional headers for caching. We also allow MIME type overrides and handle aborts. This ties into the earlier setup and leads into the completion and callback logic next.

```javascript
						match = responseHeaders[ key.toLowerCase() ];
					}
					return match === undefined ? null : match;
				},

				// Overrides response content-type header
				overrideMimeType: function( type ) {
					if ( !state ) {
						s.mimeType = type;
					}
					return this;
				},

				// Cancel the request
				abort: function( statusText ) {
					statusText = statusText || "abort";
					if ( transport ) {
						transport.abort( statusText );
					}
					done( 0, statusText );
					return this;
				}
			};

		// Callback for when everything is done
		// It is defined here because jslint complains if it is declared
		// at the end of the function (which would be more logical and readable)
		function done( status, nativeStatusText, responses, headers ) {

			// Called once
			if ( state === 2 ) {
				return;
			}

			// State is "done" now
			state = 2;

			// Clear timeout if it exists
			if ( timeoutTimer ) {
				clearTimeout( timeoutTimer );
			}

			// Dereference transport for early garbage collection
			// (no matter how long the jqXHR object will be used)
			transport = undefined;

			// Cache response headers
			responseHeadersString = headers || "";

			// Set readyState
			jqXHR.readyState = status > 0 ? 4 : 0;

			var isSuccess,
				success,
				error,
				statusText = nativeStatusText,
				response = responses ? ajaxHandleResponses( s, jqXHR, responses ) : undefined,
				lastModified,
				etag;

			// If successful, handle type chaining
			if ( status >= 200 && status < 300 || status === 304 ) {

				// Set the If-Modified-Since and/or If-None-Match header, if in ifModified mode.
				if ( s.ifModified ) {

					if ( ( lastModified = jqXHR.getResponseHeader( "Last-Modified" ) ) ) {
						jQuery.lastModified[ ifModifiedKey ] = lastModified;
					}
					if ( ( etag = jqXHR.getResponseHeader( "Etag" ) ) ) {
						jQuery.etag[ ifModifiedKey ] = etag;
					}
				}

				// If not modified
				if ( status === 304 ) {

					statusText = "notmodified";
					isSuccess = true;

				// If we have data
				} else {

					try {
						success = ajaxConvert( s, response );
						statusText = "success";
						isSuccess = true;
					} catch(e) {
						// We have a parsererror
						statusText = "parsererror";
						error = e;
					}
				}
			} else {
				// We extract error from statusText
				// then normalize statusText and status for non-aborts
				error = statusText;
				if ( !statusText || status ) {
					statusText = "error";
					if ( status < 0 ) {
						status = 0;
					}
				}
			}

			// Set data for the fake xhr object
			jqXHR.status = status;
			jqXHR.statusText = "" + ( nativeStatusText || statusText );

			// Success/Error
			if ( isSuccess ) {
				deferred.resolveWith( callbackContext, [ success, statusText, jqXHR ] );
			} else {
				deferred.rejectWith( callbackContext, [ jqXHR, statusText, error ] );
			}

			// Status-dependent callbacks
			jqXHR.statusCode( statusCode );
			statusCode = undefined;

			if ( fireGlobals ) {
				globalEventContext.trigger( "ajax" + ( isSuccess ? "Success" : "Error" ),
						[ jqXHR, s, isSuccess ? success : error ] );
			}

			// Complete
			completeDeferred.fireWith( callbackContext, [ jqXHR, statusText ] );

			if ( fireGlobals ) {
				globalEventContext.trigger( "ajaxComplete", [ jqXHR, s ] );
				// Handle the global AJAX counter
				if ( !( --jQuery.active ) ) {
					jQuery.event.trigger( "ajaxStop" );
				}
			}
		}

		// Attach deferreds
		deferred.promise( jqXHR );
		jqXHR.success = jqXHR.done;
		jqXHR.error = jqXHR.fail;
		jqXHR.complete = completeDeferred.add;

		// Status-dependent callbacks
		jqXHR.statusCode = function( map ) {
			if ( map ) {
				var tmp;
				if ( state < 2 ) {
					for ( tmp in map ) {
						statusCode[ tmp ] = [ statusCode[tmp], map[tmp] ];
					}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="7426">

---

This part sets up <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7441:9:11" line-data="		// Determine if a cross-domain request is in order">`cross-domain`</SwmToken> detection, applies prefilters for extensibility, and manages global AJAX events like <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7475:9:9" line-data="			jQuery.event.trigger( &quot;ajaxStart&quot; );">`ajaxStart`</SwmToken> and <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7555:7:7" line-data="				globalEventContext.trigger( &quot;ajaxSend&quot;, [ jqXHR, s ] );">`ajaxSend`</SwmToken>. It connects the header setup from before to the actual request logic and event triggers.

```javascript
					tmp = map[ jqXHR.status ];
					jqXHR.then( tmp, tmp );
				}
			}
			return this;
		};

		// Remove hash character (#7531: and string promotion)
		// Add protocol if not provided (#5866: IE7 issue with protocol-less urls)
		// We also use the url parameter if available
		s.url = ( ( url || s.url ) + "" ).replace( rhash, "" ).replace( rprotocol, ajaxLocParts[ 1 ] + "//" );

		// Extract dataTypes list
		s.dataTypes = jQuery.trim( s.dataType || "*" ).toLowerCase().split( rspacesAjax );

		// Determine if a cross-domain request is in order
		if ( s.crossDomain == null ) {
			parts = rurl.exec( s.url.toLowerCase() );
			s.crossDomain = !!( parts &&
				( parts[ 1 ] != ajaxLocParts[ 1 ] || parts[ 2 ] != ajaxLocParts[ 2 ] ||
					( parts[ 3 ] || ( parts[ 1 ] === "http:" ? 80 : 443 ) ) !=
						( ajaxLocParts[ 3 ] || ( ajaxLocParts[ 1 ] === "http:" ? 80 : 443 ) ) )
			);
		}

		// Convert data if not already a string
		if ( s.data && s.processData && typeof s.data !== "string" ) {
			s.data = jQuery.param( s.data, s.traditional );
		}

		// Apply prefilters
		inspectPrefiltersOrTransports( prefilters, s, options, jqXHR );

		// If request was aborted inside a prefiler, stop there
		if ( state === 2 ) {
			return false;
		}

		// We can fire global events as of now if asked to
		fireGlobals = s.global;

		// Uppercase the type
		s.type = s.type.toUpperCase();

		// Determine if request has content
		s.hasContent = !rnoContent.test( s.type );

		// Watch for a new set of requests
		if ( fireGlobals && jQuery.active++ === 0 ) {
			jQuery.event.trigger( "ajaxStart" );
		}

		// More options handling for requests with no content
		if ( !s.hasContent ) {

			// If data is available, append data to url
			if ( s.data ) {
				s.url += ( rquery.test( s.url ) ? "&" : "?" ) + s.data;
				// #9682: remove data so that it's not used in an eventual retry
				delete s.data;
			}

			// Get ifModifiedKey before adding the anti-cache parameter
			ifModifiedKey = s.url;

			// Add anti-cache in url if needed
			if ( s.cache === false ) {

				var ts = jQuery.now(),
					// try replacing _= if it is there
					ret = s.url.replace( rts, "$1_=" + ts );

				// if nothing was replaced, add timestamp to the end
				s.url = ret + ( ( ret === s.url ) ? ( rquery.test( s.url ) ? "&" : "?" ) + "_=" + ts : "" );
			}
		}

		// Set the correct header, if data is being sent
		if ( s.data && s.hasContent && s.contentType !== false || options.contentType ) {
			jqXHR.setRequestHeader( "Content-Type", s.contentType );
		}

		// Set the If-Modified-Since and/or If-None-Match header, if in ifModified mode.
		if ( s.ifModified ) {
			ifModifiedKey = ifModifiedKey || s.url;
			if ( jQuery.lastModified[ ifModifiedKey ] ) {
				jqXHR.setRequestHeader( "If-Modified-Since", jQuery.lastModified[ ifModifiedKey ] );
			}
			if ( jQuery.etag[ ifModifiedKey ] ) {
				jqXHR.setRequestHeader( "If-None-Match", jQuery.etag[ ifModifiedKey ] );
			}
		}

		// Set the Accepts header for the server, depending on the dataType
		jqXHR.setRequestHeader(
			"Accept",
			s.dataTypes[ 0 ] && s.accepts[ s.dataTypes[0] ] ?
				s.accepts[ s.dataTypes[0] ] + ( s.dataTypes[ 0 ] !== "*" ? ", " + allTypes + "; q=0.01" : "" ) :
				s.accepts[ "*" ]
		);

		// Check for headers option
		for ( i in s.headers ) {
			jqXHR.setRequestHeader( i, s.headers[ i ] );
		}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="7532">

---

Here we run <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7533:7:7" line-data="		if ( s.beforeSend &amp;&amp; ( s.beforeSend.call( callbackContext, jqXHR, s ) === false || state === 2 ) ) {">`beforeSend`</SwmToken> for last-minute changes or aborts, and wire up the success, error, and complete callbacks. This is the final prep before the transport actually sends the request.

```javascript
		// Allow custom headers/mimetypes and early abort
		if ( s.beforeSend && ( s.beforeSend.call( callbackContext, jqXHR, s ) === false || state === 2 ) ) {
				// Abort if not done already
				jqXHR.abort();
				return false;

		}

		// Install callbacks on deferreds
		for ( i in { success: 1, error: 1, complete: 1 } ) {
			jqXHR[ i ]( s[ i ] );
		}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="7545">

---

Finally, the function picks a transport, sends the request, sets up timeout handling, and returns the <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" pos="7546:17:17" line-data="		transport = inspectPrefiltersOrTransports( transports, s, options, jqXHR );">`jqXHR`</SwmToken> object. This lets the caller track the request, hook into callbacks, or abort if needed.

```javascript
		// Get transport
		transport = inspectPrefiltersOrTransports( transports, s, options, jqXHR );

		// If no transport, we auto-abort
		if ( !transport ) {
			done( -1, "No Transport" );
		} else {
			jqXHR.readyState = 1;
			// Send global event
			if ( fireGlobals ) {
				globalEventContext.trigger( "ajaxSend", [ jqXHR, s ] );
			}
			// Timeout
			if ( s.async && s.timeout > 0 ) {
				timeoutTimer = setTimeout( function(){
					jqXHR.abort( "timeout" );
				}, s.timeout );
			}

			try {
				state = 1;
				transport.send( requestHeaders, done );
			} catch (e) {
				// Propagate exception as error if not done
				if ( state < 2 ) {
					done( -1, e );
				// Simply rethrow otherwise
				} else {
					throw e;
				}
			}
		}

		return jqXHR;
	},
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
