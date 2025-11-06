---
title: Java Utility Classes Overview
---
# Overview of Java Utility Classes

In the Java portion of the project, utility classes, commonly referred to as utils, provide reusable methods that support a variety of common tasks throughout the application. These classes centralize frequently used operations such as session handling, locale management, email processing, page construction, and file or image manipulation. By consolidating these functions, utils reduce code duplication and enhance maintainability.

# Purpose and Benefits of Utility Classes

Utility classes serve to standardize and encapsulate common operations, making it easier for developers to implement business logic without rewriting boilerplate code. For example, the BeanUtils class offers methods for Java bean manipulation, including creating new instances, while EmailUtils and <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" pos="91:14:14" line-data="		home.setLabel(messages.getMessage(Constants.HOME_MENU_KEY, LocaleUtils.getLocale(language)));">`LocaleUtils`</SwmToken> focus on domain-specific tasks like email handling and locale determination. This modular approach promotes cleaner, more maintainable code.

# Using Utility Classes in the Codebase

Utility classes are typically designed with static methods, allowing developers to invoke their functionality directly without creating instances. For instance, to manipulate Java beans, one would call static methods from BeanUtils. Similarly, EmailUtils provides static methods for email-related operations. This usage pattern encourages code reuse and keeps the core business logic uncluttered by utility concerns.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" line="63">

---

<SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" pos="26:4:4" line-data="public class BreadcrumbsUtils {">`BreadcrumbsUtils`</SwmToken> is a utility class that constructs navigation breadcrumbs by processing category lineage information. It generates a breadcrumb trail reflecting the hierarchical path of categories, which enhances user navigation by clearly showing the current location within the category structure. This utility exemplifies how domain-specific logic can be encapsulated in a reusable helper class.

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

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" line="8">

---

The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="6:4:4" line-data="public class PageBuilderUtils {">`PageBuilderUtils`</SwmToken> class includes static utility methods such as <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="8:7:7" line-data="	public static String build404(MerchantStore store) {">`build404`</SwmToken>, which constructs the path to a 404 error page template. This method accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="8:9:9" line-data="	public static String build404(MerchantStore store) {">`MerchantStore`</SwmToken> object and returns a string combining a base path with the store's template name, centralizing the logic for generating consistent 404 page paths across the application.

```java
	public static String build404(MerchantStore store) {
		return new StringBuilder().append(ControllerConstants.Tiles.Pages.notFound).append(".").append(store.getStoreTemplate()).toString();
	}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" line="16">

---

<SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/BreadcrumbsUtils.java" pos="91:14:14" line-data="		home.setLabel(messages.getMessage(Constants.HOME_MENU_KEY, LocaleUtils.getLocale(language)));">`LocaleUtils`</SwmToken> provides overloaded static methods named <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="16:7:7" line-data="	public static Locale getLocale(Language language) {">`getLocale`</SwmToken> to facilitate locale determination based on different inputs. One variant accepts a Language object and returns a Locale constructed from its language code. Another variant accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="28:9:9" line-data="	public static Locale getLocale(MerchantStore store) {">`MerchantStore`</SwmToken> object and attempts to find a Locale matching the store's currency code, defaulting to a predefined locale if no match is found. These methods support consistent locale management for language and currency formatting throughout the application.

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

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
