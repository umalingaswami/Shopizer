---
title: Getting started with Java Utils
---
# Overview of Java Utils

In the codebase, Java Utils are collections of utility classes designed to provide reusable methods that support a variety of common operations throughout the application. These utility classes are grouped under the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" pos="1:8:8" line-data="package com.salesmanager.web.utils;">`utils`</SwmToken> package and serve to simplify complex or repetitive tasks, thereby promoting code reuse and maintainability.

Each utility class focuses on a specific domain or technical concern, such as bean manipulation, session management, localization, file path handling, email processing, or date operations. This modular organization helps keep the core business logic clean by delegating common technical operations to dedicated helper classes.

# Purpose and Benefits of Utils

The primary purpose of these utility classes is to reduce code duplication and improve maintainability by centralizing helper functions. By invoking static methods from these classes, developers can perform routine tasks efficiently without cluttering the main business logic with technical details.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" line="63">

---

For example, the `BeanUtils` class provides methods related to Java bean manipulation, such as creating new instances of beans. Similarly, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" pos="26:4:4" line-data="public class BreadcrumbsUtils {">`BreadcrumbsUtils`</SwmToken> manages category paths by leveraging category lineage information, simplifying the construction of breadcrumb navigation elements through methods that handle hierarchical category relationships.

```java
			//category path - use lineage
			for(Category c : categories) {
				BreadcrumbItem categoryBreadcrump = new BreadcrumbItem();
				categoryBreadcrump.setItemType(BreadcrumbItemType.CATEGORY);
				categoryBreadcrump.setLabel(c.getDescription().getName());
				categoryBreadcrump.setUrl(FilePathUtils.buildCategoryUrl(store, contextPath, c.getDescription().getSeUrl()));
				items.add(categoryBreadcrump);
			}
			
			breadCrumb.setUrlRefContent(buildBreadCrumb(ids));
			
		//}
		


		breadCrumb.setBreadCrumbs(items);
		breadCrumb.setItemType(BreadcrumbItemType.CATEGORY);
		
		
		return breadCrumb;
	}
	
	
	public Breadcrumb buildProductBreadcrumb(String refContent, ReadableProduct productClicked, MerchantStore store, Language language, String contextPath) throws Exception {
		
		/** Rebuild breadcrumb **/
		BreadcrumbItem home = new BreadcrumbItem();
		home.setItemType(BreadcrumbItemType.HOME);
		home.setLabel(messages.getMessage(Constants.HOME_MENU_KEY, LocaleUtils.getLocale(language)));
		home.setUrl(FilePathUtils.buildStoreUri(store, contextPath) + Constants.SHOP_URI);

		Breadcrumb breadCrumb = new Breadcrumb();
		breadCrumb.setLanguage(language);
		
		List<BreadcrumbItem> items = new ArrayList<BreadcrumbItem>();
		items.add(home);
		
		if(!StringUtils.isBlank(refContent)) {

			List<String> categoryIds = parseBreadCrumb(refContent);
			List<Long> ids = new ArrayList<Long>();
			for(String c : categoryIds) {
				ids.add(Long.parseLong(c));
			}
			
			
			List<Category> categories = categoryService.listByIds(store, ids, language);
			
			//category path - use lineage
```

---

</SwmSnippet>

The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" pos="26:4:4" line-data="public class BreadcrumbsUtils {">`BreadcrumbsUtils`</SwmToken> class contains methods that facilitate building breadcrumb navigation by processing category hierarchies, which is essential for user-friendly navigation in the storefront.

# Utils Endpoints and Specific Utilities

Beyond general utility classes, the codebase includes utility endpoints that provide reusable static methods for common tasks such as page building and locale management.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" line="8">

---

For instance, the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="6:4:4" line-data="public class PageBuilderUtils {">`PageBuilderUtils`</SwmToken> class includes the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="8:7:7" line-data="	public static String build404(MerchantStore store) {">`build404`</SwmToken> method, which constructs the path to a 404 error page template. This method takes a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="8:9:9" line-data="	public static String build404(MerchantStore store) {">`MerchantStore`</SwmToken> object as input and appends the store's template name to a base path defined in <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="9:11:11" line-data="		return new StringBuilder().append(ControllerConstants.Tiles.Pages.notFound).append(&quot;.&quot;).append(store.getStoreTemplate()).toString();">`ControllerConstants`</SwmToken>. Centralizing this logic ensures consistent error page handling across the application.

```java
	public static String build404(MerchantStore store) {
		return new StringBuilder().append(ControllerConstants.Tiles.Pages.notFound).append(".").append(store.getStoreTemplate()).toString();
	}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" line="16">

---

Similarly, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" pos="91:14:14" line-data="		home.setLabel(messages.getMessage(Constants.HOME_MENU_KEY, LocaleUtils.getLocale(language)));">`LocaleUtils`</SwmToken> provides overloaded <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="16:7:7" line-data="	public static Locale getLocale(Language language) {">`getLocale`</SwmToken> methods to create Java <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="16:5:5" line-data="	public static Locale getLocale(Language language) {">`Locale`</SwmToken> objects based on different inputs. One method accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="16:9:9" line-data="	public static Locale getLocale(Language language) {">`Language`</SwmToken> object and returns a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="16:5:5" line-data="	public static Locale getLocale(Language language) {">`Locale`</SwmToken> using the language code. Another accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="28:9:9" line-data="	public static Locale getLocale(MerchantStore store) {">`MerchantStore`</SwmToken> and attempts to find a matching <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="16:5:5" line-data="	public static Locale getLocale(Language language) {">`Locale`</SwmToken> based on the store's currency code by iterating over available locales, returning a default locale if no match is found. These utilities facilitate locale-sensitive operations such as formatting and localization.

```java
	public static Locale getLocale(Language language) {
		
		return new Locale(language.getCode());
		
	}
	
	/**
	 * Creates a Locale object for currency format only with country code
	 * This method ignoes the language
	 * @param store
	 * @return
	 */
	public static Locale getLocale(MerchantStore store) {
		
		Locale defaultLocale = com.salesmanager.core.constants.Constants.DEFAULT_LOCALE;
		Locale[] locales = Locale.getAvailableLocales();
		for(int i = 0; i< locales.length; i++) {
			Locale l = locales[i];
			try {
				if(l.getISO3Country().equals(store.getCurrency().getCode())) {
					defaultLocale = l;
					break;
				}
			} catch(Exception e) {
				LOGGER.error("An error occured while getting ISO code for locale " + l.toString());
			}
		}
		
		return defaultLocale;
		
	}
```

---

</SwmSnippet>

# Summary

Overall, the Java Utils in this codebase provide a structured and centralized approach to handling common technical operations. By leveraging these utility classes, developers can maintain cleaner business logic, improve code reuse, and ensure consistency across various application features.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
