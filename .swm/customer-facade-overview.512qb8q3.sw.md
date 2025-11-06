---
title: Customer Facade Overview
---
# Overview of Customer Facade

The Customer Facade serves as an abstraction layer between the Controller and the Service layer in the application. It acts as the primary entry point for customer-related operations, simplifying interactions by coordinating multiple underlying services.

This facade encapsulates complex workflows by handling pre-processing and post-processing around service calls. This design reduces the complexity in controller classes, promoting cleaner and more maintainable code.

# Facade Design Pattern

The facade pattern provides a simplified interface to a set of interfaces within a subsystem. It hides the complexity of the underlying system and exposes a cleaner, easier-to-use API. In this context, the Customer Facade abstracts the interactions with various services involved in customer management.

# Purpose and Benefits

Using the Customer Facade decouples the Controller layer from the Service layer by centralizing business logic and service interactions. This separation of concerns makes the codebase easier to maintain and extend, as controllers do not need to manage multiple service calls directly.

# Implementation Details

The facade is implemented through the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="78:8:8" line-data="public class CustomerFacadeImpl implements CustomerFacade">`CustomerFacade`</SwmToken> interface and its implementation class <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="78:4:4" line-data="public class CustomerFacadeImpl implements CustomerFacade">`CustomerFacadeImpl`</SwmToken>. This implementation coordinates services such as <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="32:14:14" line-data="import com.salesmanager.core.business.customer.service.CustomerService;">`CustomerService`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="45:14:14" line-data="import com.salesmanager.core.business.shoppingcart.service.ShoppingCartService;">`ShoppingCartService`</SwmToken>, <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="26:16:16" line-data="import com.salesmanager.core.business.catalog.product.service.PricingService;">`PricingService`</SwmToken>, and <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="46:14:14" line-data="import com.salesmanager.core.business.system.service.EmailService;">`EmailService`</SwmToken> to perform customer-related operations.

# Key Functionalities

The Customer Facade provides methods to fetch customer data by username and store, merge shopping carts during authentication, register new customers, update addresses, and authenticate customers. These methods internally call multiple services to fulfill the requested operations, handling any necessary pre- and post-processing.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="145">

---

One of the facade's endpoints fetches customer data based on a unique username within a specific store and language context. It acts as a bridge to the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="32:14:14" line-data="import com.salesmanager.core.business.customer.service.CustomerService;">`CustomerService`</SwmToken> to retrieve the customer entity and then processes it to return a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="146:3:3" line-data="    public CustomerEntity getCustomerDataByUserName( final String userName, final MerchantStore store, final Language language ) throws Exception">`CustomerEntity`</SwmToken> object suitable for the controller layer. This method ensures username uniqueness per store and manages necessary data conversions and exception handling.

```java
    @Override
    public CustomerEntity getCustomerDataByUserName( final String userName, final MerchantStore store, final Language language ) throws Exception
    {
        LOG.info( "Fetching customer with userName" +userName);
        Customer customer=customerService.getByNick( userName );

        if(customer !=null){
            LOG.info( "Found customer, converting to CustomerEntity");
            try{
            CustomerEntityPopulator customerEntityPopulator=new CustomerEntityPopulator();
            return customerEntityPopulator.populate( customer, store, language ); //store, language

            }
            catch(ConversionException ex){
                LOG.error( "Error while converting Customer to CustomerEntity", ex );
                throw new Exception(ex);
            }
        }

        return null;

    }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="172">

---

Another important method in the facade merges a customer's shopping cart with any existing session cart during authentication. This method coordinates with <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="45:14:14" line-data="import com.salesmanager.core.business.shoppingcart.service.ShoppingCartService;">`ShoppingCartService`</SwmToken> and <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="44:14:14" line-data="import com.salesmanager.core.business.shoppingcart.service.ShoppingCartCalculationService;">`ShoppingCartCalculationService`</SwmToken> to combine cart items, recalculate totals, and convert the result into a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="173:3:3" line-data="    public ShoppingCartData mergeCart( final Customer customerModel, final String sessionShoppingCartId ,final MerchantStore store,final Language language)">`ShoppingCartData`</SwmToken> object. By encapsulating this complex logic, the facade simplifies the controller's responsibilities.

```java
    @Override
    public ShoppingCartData mergeCart( final Customer customerModel, final String sessionShoppingCartId ,final MerchantStore store,final Language language)
        throws Exception
    {

        LOG.debug( "Starting merge cart process" );
        if(customerModel != null){
            ShoppingCart customerCart = shoppingCartService.getByCustomer( customerModel );
            if(StringUtils.isNotBlank( sessionShoppingCartId )){
	            ShoppingCart sessionShoppingCart = shoppingCartService.getByCode( sessionShoppingCartId, store );
	            if(sessionShoppingCart != null){
	               if(customerCart == null){
	            	   if(sessionShoppingCart.getCustomerId()==null) {//saved shopping cart does not belong to a customer
		                   LOG.debug( "Not able to find any shoppingCart with current customer" );
		                   //give it to the customer
		                   sessionShoppingCart.setCustomerId( customerModel.getId() );
		                   shoppingCartService.saveOrUpdate( sessionShoppingCart );
		                   customerCart =shoppingCartService.getById( sessionShoppingCart.getId(), store );
		                   return populateShoppingCartData(customerCart,store,language);
	            	   } else {
	            		   return null;
	            	   }
	               }
	               else{
	                    if(sessionShoppingCart.getCustomerId()==null) {//saved shopping cart does not belong to a customer
	                    	//assign it to logged in user
	                    	LOG.debug( "Customer shopping cart as well session cart is available, merging carts" );
	                    	customerCart=shoppingCartService.mergeShoppingCarts( customerCart, sessionShoppingCart, store );
	                    	customerCart =shoppingCartService.getById( customerCart.getId(), store );
		                    return populateShoppingCartData(customerCart,store,language);
	                    } else {
	                    	if(sessionShoppingCart.getCustomerId().longValue()==customerModel.getId().longValue()) {
	                    		if(!customerCart.getShoppingCartCode().equals(sessionShoppingCart.getShoppingCartCode())) {
		                    		//merge carts
		                    		LOG.info( "Customer shopping cart as well session cart is available" );
		                    		customerCart=shoppingCartService.mergeShoppingCarts( customerCart, sessionShoppingCart, store );
		                    		customerCart =shoppingCartService.getById( customerCart.getId(), store );
		    	                    return populateShoppingCartData(customerCart,store,language);
	                    		} else {
	                    			return populateShoppingCartData(sessionShoppingCart,store,language);
	                    		}
	                    	} else {
	                    		//the saved cart belongs to another user
	                    		return null;
	                    	}
	                    }
	            	    
	                    
	              }
	            }
            }
            else{
                 if(customerCart !=null){
                     return populateShoppingCartData(customerCart,store,language);
                 }
                 return null;

            }
        }
        LOG.info( "Seems some issue with system, unable to find any customer after successful authentication" );
        return null;

    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
