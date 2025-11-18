---
title: Providing payment methods by store region
---
This document describes the flow of providing payment methods available to a merchant store based on its regional settings. The flow receives the store information as input and returns a list of payment methods filtered by their applicability to the store's country or global availability.

# Filtering Available Payment Modules by Store Region

This section filters available payment modules based on the store's region to ensure only applicable payment methods are presented.

| Category       | Rule Name                      | Description                                                                                                                            |
| -------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Region-based payment filtering | Only payment modules that include the store's country in their region list or have a wildcard '\*' region are available for selection. |
| Business logic | Wildcard region applicability  | If a payment module's region list contains the wildcard '\*', it is considered applicable to all store regions.                        |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java" line="75">

---

The flow starts by getting all payment modules and filtering them by whether their regions include the store's country or the wildcard '\*', so only applicable payment methods are returned.

```java
	public List<IntegrationModule> getPaymentMethods(MerchantStore store) throws ServiceException {
		
		List<IntegrationModule> modules =  moduleConfigurationService.getIntegrationModules(PAYMENT_MODULES);
```

---

</SwmSnippet>

## Parsing and Enriching Payment Module Configurations

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start: Get integration modules for given module"] --> node2{"Modules in cache?"}
    node2 -->|"Yes"| node3["Return cached modules"]
    node2 -->|"No"| node4["Fetch modules from database"]
    node4 --> subgraph loop1["For each integration module"]
        node5{"Module has regions?"}
        node5 -->|"Yes"| node6["Parse and set regions"]
        node5 -->|"No"| node8
        node6 --> node7{"Module has details?"}
        node8 --> node7
        node7 -->|"Yes"| node9["Parse and set details"]
        node7 -->|"No"| node10
        node9 --> node11{"Module has configuration JSON?"}
        node10 --> node11
        node11 -->|"Yes"| node12["Parse and set module configurations"]
        node11 -->|"No"| node13
        node12 --> node14["End module processing"]
        node13 --> node14
        node14 --> node5
    end
    loop1 --> node15["Cache modules"]
    node15 --> node16["Return modules"]

    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:50:51"
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:57:58"
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:57:58"
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:59:60"
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:63:64"
    click node6 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:65:70"
    click node7 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:74:75"
    click node8 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:75:76"
    click node9 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:79:80"
    click node10 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:80:81"
    click node11 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:86:87"
    click node12 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:91:120"
    click node13 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:120:121"
    click node14 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:121:126"
    click node15 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:127:128"
    click node16 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java:133:134"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section handles retrieving and enriching payment integration modules by parsing JSON configurations and caching the results for efficient reuse.

| Category       | Rule Name                                 | Description                                                                                                                            |
| -------------- | ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Business logic | Parse regions                             | If a module has a regions JSON string, parse it into a set of region identifiers associated with the module.                           |
| Business logic | Parse configuration details               | If a module has configuration details as a JSON string, parse it into a key-value map of detail properties.                            |
| Business logic | Parse environment-specific configurations | If a module has a configuration JSON array, parse each entry into environment-specific ModuleConfig objects keyed by environment name. |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/ModuleConfigurationServiceImpl.java" line="50">

---

In `getIntegrationModules`, the flow starts by trying to get cached modules. If not found, it fetches from the DAO, then parses JSON strings in each module to populate region sets, detail maps, and environment-specific configurations. This enriches the modules with structured data for later use.

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

After parsing and caching, the function returns the list of enriched IntegrationModule objects. If an error occurs, it logs it and returns whatever it has, possibly null.

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

## Filtering Modules by Store Country and Wildcard Region

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start getPaymentMethods"] --> node2["Initialize empty list of payment modules"]
    node2 --> subgraph loop1["For each payment module"]
        node3{"Is module available for store's country ISO code or globally ('*')?"}
        click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:78:86"
        node3 -->|"Yes"| node4["Add module to return list"]
        click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:84:85"
        node3 -->|"No"| node6["Next module"]
        node4 --> node6
    end
    node6 --> node7["Return filtered payment modules for the store"]
    click node7 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java:78:86"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/payments/service/PaymentServiceImpl.java" line="78">

---

After returning from `getIntegrationModules`, `getPaymentMethods` filters modules by checking if the store's country ISO code or the wildcard '\*' is in the module's region set. It assumes the store's country and ISO code are valid to avoid errors.

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
