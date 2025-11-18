---
title: Back Button UI Component with TabSet and History Management
---
# introduction

This document explains the design and implementation of a back button UI component integrated with a <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="14:1:1" line-data="	TabSet initialization example">`TabSet`</SwmToken> and history management. We will cover:

1. How the component manages multiple forms within tabs.
2. How it tracks and restores tab selection state using browser history.
3. How it prevents data loss warnings on tab changes.

# forms inside tabs

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" line="30">

---

The component defines three separate <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="30:0:0" line-data="DynamicForm.create({">`DynamicForm`</SwmToken> instances, each representing a different set of input fields grouped by context: personal names, phone numbers, and address details. Each form has an <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="34:1:1" line-data="    itemChange : function () {">`itemChange`</SwmToken> handler that sets a page unload warning to prevent accidental data loss if the user tries to leave with unsaved changes. This modular form setup allows each tab to encapsulate its own data and validation logic.

```html
DynamicForm.create({
    ID:"pane1",
    autoDraw:false,
    titleOrientation:"top",
    itemChange : function () {
        Page.setUnloadMessage("Exiting the page now will lose changes");
    },
    fields:[
        {name:"firstName", title:"First Name"},
        {name:"lastName", title:"Last Name"}
    ]
});

DynamicForm.create({
    ID:"pane2",
    autoDraw:false,
    titleOrientation:"top",
    itemChange : function () {
        Page.setUnloadMessage("Exiting the page now will lose changes");
    },
    fields:[
        {name:"officeNumber", title:"Office Number"},
        {name:"mobileNumber", title:"Mobile Number"}
    ]
});

DynamicForm.create({
    ID:"pane3",
    autoDraw:false,
    titleOrientation:"top",
    itemChange : function () {
        Page.setUnloadMessage("Exiting the page now will lose changes");
    },
    fields:[
        {name:"address", title:"Street Address"},
        {name:"city", title:"City"},
        {name:"state", title:"State"},
        {name:"zip", title:"Zip"}
    ]
});
```

---

</SwmSnippet>

# tabset creation and tab management

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" line="88">

---

A <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="93:0:0" line-data="TabSet.create({">`TabSet`</SwmToken> is created with three tabs, each linked to one of the forms. The <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="93:0:0" line-data="TabSet.create({">`TabSet`</SwmToken> is configured with <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="99:1:1" line-data="    rememberHistory : true,">`rememberHistory`</SwmToken>`: true` to enable integration with browser history. It defines two key methods:

- <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="84:3:3" line-data="        tabSet.jumpToTab(isc.History.getCurrentHistoryId() || 0);">`jumpToTab`</SwmToken>`(`<SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="89:11:11" line-data="    // the id is the tabNum and null is initial state - which is the first tab.">`tabNum`</SwmToken>`)`: programmatically selects a tab without triggering history updates, used when restoring state.
- <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="106:1:1" line-data="    tabSelected : function (tabNum) {">`tabSelected`</SwmToken>`(`<SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="89:11:11" line-data="    // the id is the tabNum and null is initial state - which is the first tab.">`tabNum`</SwmToken>`)`: triggered on user tab selection, adds a new history entry unless the selection was programmatic or the page is not fully loaded. This setup ensures that user tab changes are tracked in the browser history, enabling back/forward navigation between tabs.

```html
function historyCallback(id) {
    // the id is the tabNum and null is initial state - which is the first tab.
    tabSet.jumpToTab(id == null ? 0 : id);
}

TabSet.create({
	ID:"tabSet",
	top:50,
	left:50,
	width:600,
	height:400,
    rememberHistory : true,
    jumpToTab : function (tabNum) {
        this.noHistory = true;
        // convert string to number
        this.selectTab(new Number(tabNum));
        this.noHistory = false;
    },
    tabSelected : function (tabNum) {
        if (!this.noHistory && isc.Page.isLoaded()) isc.History.addHistoryEntry(tabNum);
    },
	tabs:[{title:"red", pane:pane1, width:70},
		  {title:"green", pane:pane2, width:70},
		  {title:"blue", pane:pane3, width:70}]
});
```

---

</SwmSnippet>

# history integration and state restoration

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" line="72">

---

The component registers a global history callback <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="73:7:10" line-data="isc.History.registerCallback(&quot;historyCallback(id)&quot;);">`historyCallback(id)`</SwmToken> that is called whenever the browser history changes. This callback selects the tab corresponding to the history entry. On page load, the <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="76:12:14" line-data="isc.Page.setEvent(&quot;load&quot;, &quot;restoreTabSetState()&quot;);">`restoreTabSetState()`</SwmToken> function checks if there is existing history state; if not, it attempts to restore the tab from the URL or defaults to the first tab. This mechanism supports bookmarking and direct navigation to a specific tab state.

```html
// whenever history is navigated, call this callback
isc.History.registerCallback("historyCallback(id)");

// on page load, restore tabset state
isc.Page.setEvent("load", "restoreTabSetState()");

function restoreTabSetState() {
    isc.Log.logWarn("restoring state");
    // if we have history state, our callback will fire.  Otherwise, we need to inspect the URL
    // to see if there's history ID in there - this is what happens when the user bookmarks one
    // of the history URLs, closes the browser, opens a new one and then navigates to the bookmark.
    if (!isc.History.haveHistoryState()) {
        tabSet.jumpToTab(isc.History.getCurrentHistoryId() || 0);
    }
}
```

---

</SwmSnippet>

# external dependencies and setup

<SwmSnippet path="/shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" line="1">

---

The component loads multiple Isomorphic <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="10:10:10" line-data="	&lt;SCRIPT SRC=../../isomorphic/skins/SmartClient/load_skin.js&gt;&lt;/SCRIPT&gt;">`SmartClient`</SwmToken> modules required for UI controls, history management, and data binding. The HTML structure includes a header and sets up the environment for the <SwmToken path="shopizer/sm-shop/src/main/webapp/resources/smart-client/components/components/backButton.html" pos="10:10:10" line-data="	&lt;SCRIPT SRC=../../isomorphic/skins/SmartClient/load_skin.js&gt;&lt;/SCRIPT&gt;">`SmartClient`</SwmToken> widgets to render properly.

```html
<HTML><HEAD>
	<SCRIPT>var isomorphicDir="../../isomorphic/";</SCRIPT>
    <SCRIPT SRC=../../isomorphic/system/modules/ISC_History.js></SCRIPT>
    <SCRIPT SRC=../../isomorphic/system/modules/ISC_Core.js></SCRIPT>
    <SCRIPT SRC=../../isomorphic/system/modules/ISC_Foundation.js></SCRIPT>
    <SCRIPT SRC=../../isomorphic/system/modules/ISC_Containers.js></SCRIPT>
    <SCRIPT SRC=../../isomorphic/system/modules/ISC_Grids.js></SCRIPT>
    <SCRIPT SRC=../../isomorphic/system/modules/ISC_Forms.js></SCRIPT>
    <SCRIPT SRC=../../isomorphic/system/modules/ISC_DataBinding.js></SCRIPT>
	<SCRIPT SRC=../../isomorphic/skins/SmartClient/load_skin.js></SCRIPT>
</HEAD><BODY BGCOLOR='papayawhip' MARGINHEIGHT=0 MARGINWIDTH=0 LEFTMARGIN=0 TOPMARGIN=0>
<TABLE WIDTH=100% CELLSPACING=0 CELLPADDING=5 BORDER=0><TR><TD CLASS=pageHeader BGCOLOR=WHITE>

	TabSet initialization example

</TD><TD CLASS=pageHeader ALIGN=RIGHT BGCOLOR=WHITE>

	Isomorphic SmartClient SDK

</TD></TR></TABLE><TABLE WIDTH=100% CELLSPACING=0 CELLPADDING=0 BORDER=0><TR>
<TD BGCOLOR=336666><IMG SRC=images/blank.gif WIDTH=1 HEIGHT=4></TD></TR></TABLE>


<!--------------------------
  Example code starts here
---------------------------->

<SCRIPT>
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
