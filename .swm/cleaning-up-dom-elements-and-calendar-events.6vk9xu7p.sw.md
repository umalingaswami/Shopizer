---
title: Cleaning Up DOM Elements and Calendar Events
---
This document describes how DOM elements and calendar events are cleaned up to keep the user interface consistent and performant. The flow determines which elements and events should be processed, removes their event handlers and cached data, and refreshes calendar views as needed.

# Preparing DOM Elements for Cleanup

This section governs which DOM elements are eligible for cleanup by filtering out elements that should not be processed, identifying their cache IDs, and preparing them for event handler removal. The rules ensure that only valid elements are cleaned, maintaining UI stability and performance.

| Category        | Rule Name                   | Description                                                                                                                 |
| --------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Data validation | NoData Element Exclusion    | DOM elements with node names listed in the noData registry must not be cleaned or have their event handlers removed.        |
| Data validation | Cache ID Requirement        | Only DOM elements with a valid cache ID are eligible for event handler removal and data cleanup.                            |
| Business logic  | Event Handler Removal       | All event handlers associated with eligible DOM elements must be removed to prevent memory leaks and ensure UI consistency. |
| Business logic  | Special Event Type Handling | Special event types must be removed using their designated removal process to ensure correct cleanup behavior.              |

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="6365">

---

In `cleanData`, we filter out elements that shouldn't be cleaned, grab their jQuery cache IDs, and prep for event handler removal. This leads into the calendar module for UI updates.

```javascript
	cleanData: function( elems ) {
		var data, id,
			cache = jQuery.cache,
			special = jQuery.event.special,
			deleteExpando = jQuery.support.deleteExpando;

		for ( var i = 0, elem; (elem = elems[i]) != null; i++ ) {
			if ( elem.nodeName && jQuery.noData[elem.nodeName.toLowerCase()] ) {
				continue;
			}

			id = elem[ jQuery.expando ];

			if ( id ) {
				data = cache[ id ];

				if ( data && data.events ) {
					for ( var type in data.events ) {
						if ( special[ type ] ) {
							jQuery.event.remove( elem, type );

						// This is a shortcut to avoid jQuery.event.remove's overhead
						} else {
							jQuery.removeEvent( elem, type, data.handle );
						}
					}

```

---

</SwmSnippet>

## Removing and Refreshing Calendar Events

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start event removal"]
    click node1 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:66:71"
    node1 --> node2{"Is there a data source for events?"}
    click node2 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:71:71"
    node2 -->|"Yes"| node3["Remove event from data source"]
    click node3 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:71:71"
    node2 -->|"No"| node4["Remove event from local data"]
    click node4 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:71:71"
    node3 --> node5["Update calendar views (day, week, month)"]
    click node5 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:66:70"
    node4 --> node5
    node5 --> node6{"Should auto-arrange events?"}
    click node6 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:69:69"
    node6 -->|"Yes"| node7["Auto-arrange calendar views"]
    click node7 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:69:69"
    node7 --> node8["Trigger event removed callback if set"]
    click node8 openCode "shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js:70:70"
    node6 -->|"No"| node8
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section governs how calendar events are removed and how the calendar UI is refreshed to ensure consistency and accuracy after an event is deleted. It ensures that all relevant views are updated and that any custom business logic related to event removal is executed.

| Category       | Rule Name                      | Description                                                                                                                                                                               |
| -------------- | ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Data source removal precedence | If a data source is configured for calendar events, the event must be removed from the data source. If no data source is configured, the event must be removed from the local data store. |
| Business logic | Calendar view consistency      | After an event is removed, all calendar views (day, week, month) that display the event must be updated to reflect its removal.                                                           |
| Business logic | Auto-arrange enforcement       | If the auto-arrange setting is enabled, the calendar must automatically rearrange events in the day and week views after an event is removed.                                             |
| Business logic | Custom removal callback        | If a custom event removal callback is configured, it must be triggered after the event is removed and the views are updated.                                                              |

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js" line="66">

---

`removeEvent` checks the event's start and end dates against day, week, and month views using repo-specific helpers. It then removes or refreshes events in those views as needed, and triggers any custom event removal logic. This keeps the calendar UI consistent after data cleanup.

```javascript
,isc.A.removeEvent=function isc_Calendar_removeEvent(_1,_2){if(_2==null)_2=true;var _3=_1[this.startDateField],_4=_1[this.endDateField];var _5=this;var _6=function(){if(_5.$53b(_3,_4)){_5.dayView.removeEvent(_1)}
if(_5.$53c(_3,_4)){_5.weekView.removeEvent(_1)}
if(_5.$53d(_3,_4)){_5.monthView.refreshEvents()}
if(_5.eventAutoArrange){if(_5.dayView)_5.dayView.refreshEvents();if(_5.weekView)_5.weekView.refreshEvents()}
if(_5.eventRemoved)_5.eventRemoved(_1)}
if(_2)this.$53e=true;if(this.dataSource){isc.DataSource.get(this.dataSource).removeData(_1,_6,{componentId:this.ID,oldValues:_1});return}else{this.data.remove(_1);_6()}}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/smart-client/system/modules/ISC_Calendar.js" line="66">

---

`removeEvent` checks which calendar views the event touches and updates them so the UI matches the data.

```javascript
,isc.A.removeEvent=function isc_Calendar_removeEvent(_1,_2){if(_2==null)_2=true;var _3=_1[this.startDateField],_4=_1[this.endDateField];var _5=this;var _6=function(){if(_5.$53b(_3,_4)){_5.dayView.removeEvent(_1)}
if(_5.$53c(_3,_4)){_5.weekView.removeEvent(_1)}
if(_5.$53d(_3,_4)){_5.monthView.refreshEvents()}
if(_5.eventAutoArrange){if(_5.dayView)_5.dayView.refreshEvents();if(_5.weekView)_5.weekView.refreshEvents()}
if(_5.eventRemoved)_5.eventRemoved(_1)}
```

---

</SwmSnippet>

## Finalizing Cleanup and Memory Management

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start cleaning DOM element data"] --> node2{"Does data.handle exist?"}
    click node1 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6392:6392"
    node2 -->|"Yes"| node3["Nullify DOM reference to prevent memory leak"]
    click node2 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6393:6395"
    click node3 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6394:6394"
    node2 -->|"No"| node4
    node3 --> node4
    node4 --> node5{"Should expando property be deleted?"}
    click node5 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6398:6403"
    node5 -->|"Yes"| node6["Delete metadata property from element"]
    click node6 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6399:6399"
    node5 -->|"No"| node7{"Can removeAttribute be used?"}
    click node7 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6401:6403"
    node7 -->|"Yes"| node8["Remove metadata attribute from element"]
    click node8 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6402:6402"
    node7 -->|"No"| node10["Delete cache entry for element"]
    node6 --> node10["Delete cache entry for element"]
    node8 --> node10["Delete cache entry for element"]
    click node10 openCode "shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js:6405:6405"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/js/bootstrap/jquery.js" line="6392">

---

After updating the calendar, `cleanData` clears out any leftover references and cached data to prevent leaks and keep things tidy.

```javascript
					// Null the DOM reference to avoid IE6/7/8 leak (#7054)
					if ( data.handle ) {
						data.handle.elem = null;
					}
				}

				if ( deleteExpando ) {
					delete elem[ jQuery.expando ];

				} else if ( elem.removeAttribute ) {
					elem.removeAttribute( jQuery.expando );
				}

				delete cache[ id ];
			}
		}
	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
