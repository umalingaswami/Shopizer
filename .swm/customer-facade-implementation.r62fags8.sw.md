---
title: Customer Facade Implementation
---
# Introduction

This document explains the main ideas behind the implementation of the customer facade in <SwmPath>[shopizer/…/facade/CustomerFacadeImpl.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java)</SwmPath>. The facade acts as a bridge between controllers and the service layer, handling customer-related operations and abstracting away service complexity.

We will cover:

1. Why the facade pattern is used for customer logic.
2. How customer retrieval and registration are handled.
3. The approach to shopping cart merging for authenticated customers.
4. How customer address management is implemented.
5. The rationale for authentication and group assignment.

# Why use a facade for customer logic

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="69">

---

The facade is annotated as a Spring service and implements the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" pos="78:8:8" line-data="public class CustomerFacadeImpl implements CustomerFacade">`CustomerFacade`</SwmToken> interface. This design centralizes customer-related operations, so controllers don't need to interact directly with multiple services. It also makes it easier to change business logic without affecting controller code.

```java
/**
 * Customer Facade work as an abstraction layer between Controller and Service layer.
 * It work as an entry point to service layer.
 * @author Umesh Awasthi
 *
 */

@Service("customerFacade")
//// http://stackoverflow.com/questions/17444258/how-to-use-new-passwordencoder-from-spring-security
public class CustomerFacadeImpl implements CustomerFacade
{
```

---

</SwmSnippet>

# Customer retrieval and registration

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="137">

---

Fetching customer data by username is done through the service layer, then converted to a DTO for controller use. This keeps domain models separate from API responses and allows for flexible mapping.

```java
    /**
     * Method used to fetch customer based on the username and storecode.
     * Customer username is unique to each store.
     *
     * @param userName
     * @param storeCode
     * @throws ConversionException
     */
    @Override
    public CustomerEntity getCustomerDataByUserName( final String userName, final MerchantStore store, final Language language ) throws Exception
    {
        LOG.info( "Fetching customer with userName" +userName);
        Customer customer=customerService.getByNick( userName );
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="151">

---

The conversion to a DTO is handled by a populator, which abstracts the mapping logic and allows for easier changes to the DTO structure.

```java
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
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="298">

---

Customer registration involves mapping incoming data to a domain model, persisting it, and returning a DTO. The facade ensures that all required services are used for mapping and validation, and throws a specific exception if registration fails.

```java
    @Override
    public CustomerEntity registerCustomer( final PersistableCustomer customer,final MerchantStore merchantStore, Language language )
        throws Exception
    {
       LOG.info( "Starting customer registration process.." );
        Customer customerModel= getCustomerModel(customer,merchantStore,language);
        if(customerModel == null){
            LOG.equals( "Unable to create customer in system" );
            throw new CustomerRegistrationException( "Unable to register customer" );
        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="309">

---

Persisting the customer is a separate step, which allows for pre-processing and validation before saving.

```java
        LOG.info( "About to persist customer to database." );
        customerService.saveOrUpdate( customerModel );
```

---

</SwmSnippet>

# Shopping cart merging for authenticated customers

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="177">

---

When a user logs in, the facade merges their session cart with any existing cart associated with their account. This prevents cart loss and ensures a consistent shopping experience. The logic checks ownership and merges only when appropriate, otherwise returns null to signal an error or conflict.

```java
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
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="207">

---

Actual merging is delegated to the shopping cart service, and the result is mapped to a DTO for controller use.

```java
		                    		customerCart=shoppingCartService.mergeShoppingCarts( customerCart, sessionShoppingCart, store );
		                    		customerCart =shoppingCartService.getById( customerCart.getId(), store );
		    	                    return populateShoppingCartData(customerCart,store,language);
```

---

</SwmSnippet>

# Customer address management

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="445">

---

Address retrieval checks for customer existence and throws a specific exception if not found. This prevents silent failures and makes error handling explicit.

```java
        if(customerModel == null){
            LOG.error( "Customer with ID {} does not exists..", userId);
            throw new CustomerNotFoundException( "customer with given id does not exists" ); 
        }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="450">

---

Billing and delivery addresses are handled by separate populators, which encapsulate the mapping logic and allow for future changes without touching the facade.

```java
       if(isBillingAddress){
            LOG.info( "getting billing address.." );
            CustomerBillingAddressPopulator billingAddressPopulator=new CustomerBillingAddressPopulator();
            address =billingAddressPopulator.populate( customerModel, merchantStore, merchantStore.getDefaultLanguage() );
            address.setBillingAddress( true );
            return address;
        }
```

---

</SwmSnippet>

# Authentication and group assignment

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="408">

---

Authentication is handled by building a Spring Security token with the correct authorities, based on the customer's groups and permissions. This ensures that only authorized users can access protected resources.

```java
        	Collection<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
			GrantedAuthority role = new GrantedAuthorityImpl(Constants.PERMISSION_CUSTOMER_AUTHENTICATED);//required to login
			authorities.add(role); 
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/facade/CustomerFacadeImpl.java" line="388">

---

Group assignment is handled during customer creation, ensuring that every customer has the correct default group and permissions. This is important for role-based access control and feature gating.

```java
		if(CollectionUtils.isEmpty(customer.getGroups())) {
			List<Group> groups = groupService.listGroup(GroupType.CUSTOMER);
			for(Group group : groups) {
				  if(group.getGroupName().equals(Constants.GROUP_CUSTOMER)) {
					  customer.getGroups().add(group);
				  }
			}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
