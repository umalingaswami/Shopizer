---
title: Retrieving applicable payment methods
---
This document explains the flow of retrieving payment methods for a merchant store by fetching all payment integration modules, parsing their configurations and regions, and filtering them to return only those applicable to the store's country or global region. This ensures the merchant store receives relevant payment options.

```mermaid
flowchart TD
  node1["Starting payment methods retrieval"]:::HeadingStyle
  node1 --> node2["Loading and parsing integration modules"]:::HeadingStyle
  node2 --> node3{"Module supports store's country or global region?
(Filtering modules by store region)"}:::HeadingStyle
  node3 -->|"Yes"| node4["Add module to return list
(Filtering modules by store region)"]:::HeadingStyle
  node3 -->|"No"| node5["Skip module
(Filtering modules by store region)"]:::HeadingStyle
  node4 --> node6["Return filtered payment methods
(Filtering modules by store region)"]:::HeadingStyle
  node5 --> node6
  click node1 goToHeading "Starting payment methods retrieval"
  click node2 goToHeading "Loading and parsing integration modules"
  click node3 goToHeading "Filtering modules by store region"
  click node4 goToHeading "Filtering modules by store region"
  click node5 goToHeading "Filtering modules by store region"
  click node6 goToHeading "Filtering modules by store region"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Starting payment methods retrieval

This section describes the retrieval of payment methods for a merchant store by fetching all payment-related integration modules and filtering them accordingly.

| Category       | Rule Name                       | Description                                                                                                            |
| -------------- | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Business logic | Filter payment methods by store | Only payment methods relevant to the specific merchant store, based on configurations and regions, should be returned. |
| Business logic | Return payment methods list     | The system must return a list of payment method modules that are valid and applicable for the store.                   |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java" line="75">

---

In `getPaymentMethods`, we start by fetching all payment-related integration modules using getIntegrationModules. This call is necessary because it loads the raw module data, including configurations and regions, which we need to filter and return relevant payment methods for the store.

```java
	public List<IntegrationModule> getPaymentMethods(MerchantStore store) throws ServiceException {
		
		List<IntegrationModule> modules =  moduleConfigurationService.getIntegrationModules(PAYMENT_MODULES);
```

---

</SwmSnippet>

## Loading and parsing integration modules

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Modules found in cache?"}
    node1 -->|"Yes"| node2["Return cached modules"]
    node1 -->|"No"| node3["Fetch modules from database"]
    
    subgraph loop1["For each integration module"]
        node3 --> node4{"Module has regions?"}
        node4 -->|"Yes"| node5["Parse and add regions to module"]
        node4 -->|"No"| node7
        node5 --> node7
        node7{"Module has details?"}
        node7 -->|"Yes"| node8["Parse and set details"]
        node7 -->|"No"| node10
        node8 --> node10
        node10{"Module has configuration JSON?"}
        node10 -->|"Yes"| node11["Parse and set module configurations"]
        node10 -->|"No"| node13
        node11 --> node13
        node13 --> node3
    end
    node3 --> node14["Cache modules"]
    node14 --> node15["Return modules"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section describes the loading and parsing of integration modules, including caching, fetching from the database, and parsing JSON configuration data for use in the application.

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java" line="50">

---

In `getIntegrationModules`, we first try to get the modules from cache. If not found, we fetch from the DAO. Then, for each module, we parse JSON strings from regions, configDetails, and configuration fields to populate sets and maps that the rest of the app can use. We cache the result for next calls. Note the bug where config2 overwrites config1 in ModuleConfig.

```java
	public List<IntegrationModule> getIntegrationModules(String module) {
		
		
		List<IntegrationModule> modules = null;
		try {
			
			//CacheUtils cacheUtils = CacheUtils.getInstance();
			modules = (List<IntegrationModule>) cache.getFromCache("INTEGRATION_M)" + module);
			if(modules==null) {
				modules = integrationModuleDao.getModulesConfiguration(module);
				//set json objects
				for(IntegrationModule mod : modules) {
					
					String regions = mod.getRegions();
					if(regions!=null) {
						Object objRegions=JSONValue.parse(regions); 
						JSONArray arrayRegions=(JSONArray)objRegions;
						Iterator i = arrayRegions.iterator();
						while(i.hasNext()) {
							mod.getRegionsSet().add((String)i.next());
						}
					}
					
					
					String details = mod.getConfigDetails();
					if(details!=null) {
						
						//Map objects = mapper.readValue(config, Map.class);

						Map<String,String> objDetails= (Map<String, String>) JSONValue.parse(details); 
						mod.setDetails(objDetails);

						
					}
					
					
					String configs = mod.getConfiguration();
					if(configs!=null) {
						
						//Map objects = mapper.readValue(config, Map.class);

						Object objConfigs=JSONValue.parse(configs); 
						JSONArray arrayConfigs=(JSONArray)objConfigs;
						
						Map<String,ModuleConfig> moduleConfigs = new HashMap<String,ModuleConfig>();
						
						Iterator i = arrayConfigs.iterator();
						while(i.hasNext()) {
							
							Map values = (Map)i.next();
							String env = (String)values.get("env");
		            		ModuleConfig config = new ModuleConfig();
		            		config.setScheme((String)values.get("scheme"));
		            		config.setHost((String)values.get("host"));
		            		config.setPort((String)values.get("port"));
		            		config.setUri((String)values.get("uri"));
		            		config.setEnv((String)values.get("env"));
		            		if((String)values.get("config1")!=null) {
		            			config.setConfig1((String)values.get("config1"));
		            		}
		            		if((String)values.get("config2")!=null) {
		            			config.setConfig1((String)values.get("config2"));
		            		}
		            		
		            		moduleConfigs.put(env, config);
		            		
		            		
							
						}
						
						mod.setModuleConfigs(moduleConfigs);
						

					}


				}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java" line="127">

---

We cache the fully parsed modules and return them to speed up future retrievals.

```java
				cache.putInCache(modules, "INTEGRATION_M)" + module);
			}

		} catch (Exception e) {
			LOGGER.error("getIntegrationModules()", e);
		}
		return modules;
		
		
	}
```

---

</SwmSnippet>

## Filtering modules by store region

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start getPaymentMethods"]
    node1 --> node2["Get list of all payment modules"]
    subgraph loop1["For each payment module"]
        node3{"Module supports store's country ISO code or global #quot;*#quot;?"}
        node3 -->|"Yes"| node4["Add module to return list"]
        node3 -->|"No"| node5["Skip module"]
        node4 --> node6["Next module"]
        node5 --> node6
    end
    node2 --> loop1
    node6 --> node7["Return filtered payment modules"]

    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:78:79"
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:78:79"
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:81:83"
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:84:85"
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:85:86"
    click node6 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:86:87"
    click node7 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:87:88"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java" line="78">

---

Back in `getPaymentMethods`, after getting all modules, we filter them by checking if the store's country code or a wildcard is in the module's regions set. Only matching modules are returned as available payment methods.

```java
		List<IntegrationModule> returnModules = new ArrayList<IntegrationModule>();
		
		for(IntegrationModule module : modules) {
			if(module.getRegionsSet().contains(store.getCountry().getIsoCode())
					|| module.getRegionsSet().contains("*")) {
				
				returnModules.add(module);
			}
		}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
