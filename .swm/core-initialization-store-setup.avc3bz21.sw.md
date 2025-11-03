---
title: Core Initialization Store Setup
---
# Introduction to Core Initialization Store Setup

Core Initialization Store Setup refers to the process of establishing the default merchant store and its related entities within the database during the initial startup or setup of the application. This foundational step ensures that the application has a baseline store configuration that supports further customization and operational functionality.

# Purpose of Store Initialization

The primary purpose of store initialization is to create a consistent and valid merchant store setup. This setup includes essential attributes such as country, currency, default language, zone, store name, contact details, and supported languages. Additionally, it establishes related entities like the default tax class to enable proper tax handling within the store context.

# Store Initialization Process

The store initialization is performed by the <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="190:5:7" line-data="	private void createMerchant() throws ServiceException {">`createMerchant()`</SwmToken> method. This method constructs a new <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="204:1:1" line-data="		MerchantStore store = new MerchantStore();">`MerchantStore`</SwmToken> object and assigns it predefined attributes including geographical and localization data, store identity, and communication details. After configuring these properties, the method persists the store entity using the merchant service. Following this, it creates and persists a default tax class linked to the merchant store to manage taxation effectively.

# Store Initialization in the Overall Initialization Sequence

Store initialization is a part of a comprehensive database population sequence that ensures all referenced domain entities exist before the store is created. This sequence includes creating languages, countries, zones, currencies, product types, and integration modules. By following this order, the application maintains data integrity and consistency across its core entities.

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" line="190">

---

Within the <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="40:4:4" line-data="public class InitializationDatabaseImpl implements InitializationDatabase {">`InitializationDatabaseImpl`</SwmToken> class, the `populate()` method orchestrates the entire initialization sequence. It sequentially calls methods such as <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="88:1:3" line-data="		createLanguages();">`createLanguages()`</SwmToken>, <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="89:1:3" line-data="		createCountries();">`createCountries()`</SwmToken>, <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="90:1:3" line-data="		createZones();">`createZones()`</SwmToken>, <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="91:1:3" line-data="		createCurrencies();">`createCurrencies()`</SwmToken>, <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="92:1:3" line-data="		createSubReferences();">`createSubReferences()`</SwmToken>, <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="93:1:3" line-data="		createModules();">`createModules()`</SwmToken>, and finally <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="190:5:7" line-data="	private void createMerchant() throws ServiceException {">`createMerchant()`</SwmToken>. The <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/reference/init/service/InitializationDatabaseImpl.java" pos="190:5:7" line-data="	private void createMerchant() throws ServiceException {">`createMerchant()`</SwmToken> method is responsible for setting up the default merchant store and creating the associated default tax class, thereby completing the store initialization process.

```java
	private void createMerchant() throws ServiceException {
		LOGGER.info(String.format("%s : Creating merchant ", name));
		
		Date date = new Date(System.currentTimeMillis());
		
		Language en = languageService.getByCode("en");
		Country ca = countryService.getByCode("CA");
		Currency currency = currencyService.getByCode("CAD");
		Zone qc = zoneService.getByCode("QC");
		
		List<Language> supportedLanguages = new ArrayList<Language>();
		supportedLanguages.add(en);
		
		//create a merchant
		MerchantStore store = new MerchantStore();
		store.setCountry(ca);
		store.setCurrency(currency);
		store.setDefaultLanguage(en);
		store.setInBusinessSince(date);
		store.setZone(qc);
		store.setStorename("Default store");
		store.setStorephone("888-888-8888");
		store.setCode(MerchantStore.DEFAULT_STORE);
		store.setStorecity("My city");
		store.setStoreaddress("1234 Street address");
		store.setStorepostalcode("H2H-2H2");
		store.setStoreEmailAddress("test@test.com");
		store.setDomainName("localhost:8080");
		store.setStoreTemplate("bootstrap");
		store.setLanguages(supportedLanguages);
		
		merchantService.create(store);
		
		
		TaxClass taxclass = new TaxClass(TaxClass.DEFAULT_TAX_CLASS);
		taxclass.setMerchantStore(store);
		
		taxClassService.create(taxclass);
		
		
	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
