---
title: Java Utility Classes Overview
---
# Introduction to Java Utils

In this Java project, utility classes, commonly referred to as Utils, provide reusable methods that support a variety of common operations throughout the application. These classes encapsulate frequently needed functionality such as bean manipulation, session management, email processing, and file handling.

Utils classes promote modularity and reduce code duplication by centralizing shared logic that different components of the application can use. Typically, these classes are stateless and offer static methods or factory methods to create instances when necessary.

# Purpose of Utils

The primary purpose of Utils classes is to serve as a centralized repository for reusable helper methods that perform common tasks needed across the application. This design pattern avoids repeating code and enhances maintainability by allowing multiple parts of the system to leverage the same utility logic.

# How to Use Utils

Since Utils classes are generally stateless and contain static methods, their functionality can be accessed directly without creating an instance of the class. For example, to dynamically retrieve a property value from a Java bean, you can call a method from the BeanUtils class, which simplifies reflection-based operations.

# Example: BeanUtils Class

The BeanUtils class provides methods to introspect Java beans and retrieve property values dynamically using reflection. This is particularly useful when you want to manipulate or access bean properties without hardcoding property names, enabling more flexible and generic code.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" line="8">

---

The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="8:7:7" line-data="	public static String build404(MerchantStore store) {">`build404`</SwmToken> method in the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="6:4:4" line-data="public class PageBuilderUtils {">`PageBuilderUtils`</SwmToken> class constructs the path to a 404 error page template dynamically. It accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="8:9:9" line-data="	public static String build404(MerchantStore store) {">`MerchantStore`</SwmToken> object and appends the store's template name to a base path defined in <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/PageBuilderUtils.java" pos="9:11:17" line-data="		return new StringBuilder().append(ControllerConstants.Tiles.Pages.notFound).append(&quot;.&quot;).append(store.getStoreTemplate()).toString();">`ControllerConstants.Tiles.Pages.notFound`</SwmToken>. This allows the application to serve a store-specific 404 error page.

```java
	public static String build404(MerchantStore store) {
		return new StringBuilder().append(ControllerConstants.Tiles.Pages.notFound).append(".").append(store.getStoreTemplate()).toString();
	}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" line="16">

---

The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="12:4:4" line-data="public class LocaleUtils {">`LocaleUtils`</SwmToken> class provides overloaded <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="16:7:7" line-data="	public static Locale getLocale(Language language) {">`getLocale`</SwmToken> methods to obtain locale information based on different inputs. One version accepts a Language object and returns a Locale constructed from the language code. Another version accepts a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/utils/LocaleUtils.java" pos="28:9:9" line-data="	public static Locale getLocale(MerchantStore store) {">`MerchantStore`</SwmToken> object and attempts to find a Locale matching the store's currency code by iterating over available locales. If no match is found, it returns a default locale. This functionality is essential for formatting currency and other locale-sensitive data according to the store's settings.

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
