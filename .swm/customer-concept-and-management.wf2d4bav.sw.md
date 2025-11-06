---
title: Customer Concept and Management
---
# Customer Concept Overview

In the e-commerce platform, a Customer represents the end user who interacts with the shop to browse products, place orders, and manage their account. This concept encapsulates the customer's identity and their various interactions within the system.

# Customer Role in the Application

The Customer entity plays a central role in the application by enabling personalized shopping experiences. It is used to track user sessions, manage orders, and provide access to account-specific features such as order history and profile management.

# Customer Controllers and Their Responsibilities

Customer-related operations are organized within a dedicated controller package. For example, the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" pos="69:4:4" line-data="public class CustomerAccountController extends AbstractController {">`CustomerAccountController`</SwmToken> manages requests related to viewing and updating account details, including password changes. Similarly, the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerRegistrationController.java" pos="61:4:4" line-data="public class CustomerRegistrationController extends AbstractController {">`CustomerRegistrationController`</SwmToken> handles new user sign-ups, validating input and creating new customer records.

# Security and Authorization

Access to customer-specific pages, such as the account overview and password change pages, is protected by role-based authorization. Only authenticated users with the appropriate customer role can access these features, ensuring data privacy and security.

# Dynamic View Rendering Based on Store Configuration

Controllers dynamically select view templates based on the merchant store's configuration. This design allows for customizable storefronts tailored to each merchant's branding and layout preferences, enhancing the flexibility of the platform.

# Integration with Facade and Service Layers

The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerRegistrationController.java" pos="47:16:16" line-data="import com.salesmanager.web.shop.controller.customer.facade.CustomerFacade;">`CustomerFacade`</SwmToken> serves as an abstraction layer between the controllers and the underlying service layer. It provides methods to retrieve and manipulate customer data, such as fetching a customer by username or merging shopping carts, thereby simplifying controller logic and promoting separation of concerns.

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerRegistrationController.java" line="123">

---

The registration endpoint `/shop/customer/register.html` accepts POST requests to create new customers. The controller validates the registration form, checks for existing users, and invokes the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerRegistrationController.java" pos="124:5:5" line-data="    public String registerCustomer( @Valid">`registerCustomer`</SwmToken> method of the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerRegistrationController.java" pos="47:16:16" line-data="import com.salesmanager.web.shop.controller.customer.facade.CustomerFacade;">`CustomerFacade`</SwmToken> to persist the new customer. Upon successful registration, a confirmation email is sent and the user is automatically logged in, streamlining the onboarding process.

```java
    @RequestMapping( value = "/register.html", method = RequestMethod.POST )
    public String registerCustomer( @Valid
    @ModelAttribute("customer") SecuredShopPersistableCustomer customer, BindingResult bindingResult, Model model,
                                    HttpServletRequest request, final Locale locale )
        throws Exception
    {
        MerchantStore merchantStore = (MerchantStore) request.getAttribute( Constants.MERCHANT_STORE );
        Language language = super.getLanguage(request);
        
        
        ReCaptchaImpl reCaptcha = new ReCaptchaImpl();
        reCaptcha.setPublicKey( coreConfiguration.getProperty( Constants.RECAPATCHA_PUBLIC_KEY ) );
        reCaptcha.setPrivateKey( coreConfiguration.getProperty( Constants.RECAPATCHA_PRIVATE_KEY ) );
        
        String userName = null;
        String password = null;
        
        model.addAttribute( "recapatcha_public_key", coreConfiguration.getProperty( Constants.RECAPATCHA_PUBLIC_KEY ) );
        
        if ( StringUtils.isNotBlank( customer.getRecaptcha_challenge_field() )
            && StringUtils.isNotBlank( customer.getRecaptcha_response_field() ) )
        {
            ReCaptchaResponse reCaptchaResponse =
                reCaptcha.checkAnswer( request.getRemoteAddr(), customer.getRecaptcha_challenge_field(),
                                       customer.getRecaptcha_response_field() );
            if ( !reCaptchaResponse.isValid() )
            {
                LOGGER.debug( "Captcha response does not matched" );
    			FieldError error = new FieldError("recaptcha_challenge_field","recaptcha_challenge_field",messages.getMessage("validaion.recaptcha.not.matched", locale));
    			bindingResult.addError(error);
            }

        }
        
        if ( StringUtils.isNotBlank( customer.getUserName() ) )
        {
            if ( customerFacade.checkIfUserExists( customer.getUserName(), merchantStore ) )
            {
                LOGGER.debug( "Customer with username {} already exists for this store ", customer.getUserName() );
            	FieldError error = new FieldError("userName","userName",messages.getMessage("registration.username.already.exists", locale));
            	bindingResult.addError(error);
            }
            userName = customer.getUserName();
        }
        
        
        if ( StringUtils.isNotBlank( customer.getPassword() ) &&  StringUtils.isNotBlank( customer.getCheckPassword() ))
        {
            if (! customer.getPassword().equals(customer.getCheckPassword()) )
            {
            	FieldError error = new FieldError("password","password",messages.getMessage("message.password.checkpassword.identical", locale));
            	bindingResult.addError(error);

            }
            password = customer.getPassword();
        }

        if ( bindingResult.hasErrors() )
        {
            LOGGER.debug( "found {} validation error while validating in customer registration ",
                         bindingResult.getErrorCount() );
            StringBuilder template =
                new StringBuilder().append( ControllerConstants.Tiles.Customer.register ).append( "." ).append( merchantStore.getStoreTemplate() );
            return template.toString();

        }

        @SuppressWarnings( "unused" )
        CustomerEntity customerData = null;
        try
        {
            //set user clear password
        	customer.setClearPassword(password);
        	customerData = customerFacade.registerCustomer( customer, merchantStore, language );
        }
        catch ( CustomerRegistrationException cre )
        {
            LOGGER.error( "Error while registering customer.. ", cre);
        	ObjectError error = new ObjectError("registration",messages.getMessage("registration.failed", locale));
        	bindingResult.addError(error);
            StringBuilder template =
                            new StringBuilder().append( ControllerConstants.Tiles.Customer.register ).append( "." ).append( merchantStore.getStoreTemplate() );
             return template.toString();
        }
        catch ( Exception e )
        {
            LOGGER.error( "Error while registering customer.. ", e);
        	ObjectError error = new ObjectError("registration",messages.getMessage("registration.failed", locale));
        	bindingResult.addError(error);
            StringBuilder template =
                            new StringBuilder().append( ControllerConstants.Tiles.Customer.register ).append( "." ).append( merchantStore.getStoreTemplate() );
            return template.toString();
        }
              
        /**
         * Send registration email
         */
        emailTemplatesUtils.sendRegistrationEmail( customer, merchantStore, locale, request.getContextPath() );

        /**
         * Login user
         */
        
        try {
        	
	        //refresh customer
	        Customer c = customerFacade.getCustomerByUserName(customer.getUserName(), merchantStore);
	        //authenticate
	        customerFacade.authenticate(c, userName, password);
	        super.setSessionAttribute(Constants.CUSTOMER, c, request);
	        
	        return "redirect:/shop/customer/dashboard.html";
        
        
        } catch(Exception e) {
        	LOGGER.error("Cannot authenticate user ",e);
        	ObjectError error = new ObjectError("registration",messages.getMessage("registration.failed", locale));
        	bindingResult.addError(error);
        }
        
        
        StringBuilder template =
                new StringBuilder().append( ControllerConstants.Tiles.Customer.register ).append( "." ).append( merchantStore.getStoreTemplate() );
        return template.toString();

    }
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" line="147">

---

The account page endpoint `/shop/customer/account.html` is accessible via GET requests and restricted to authenticated customers with the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" pos="146:8:8" line-data="	@PreAuthorize(&quot;hasRole(&#39;AUTH_CUSTOMER&#39;)&quot;)">`AUTH_CUSTOMER`</SwmToken> role. The <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java" pos="69:4:4" line-data="public class CustomerAccountController extends AbstractController {">`CustomerAccountController`</SwmToken> retrieves the current store and customer information, prepares the model, and returns the appropriate view template to display the customer's account details.

```java
	@RequestMapping(value="/account.html", method=RequestMethod.GET)
	public String displayCustomerAccount(Model model, HttpServletRequest request, HttpServletResponse response) throws Exception {
		

	    MerchantStore store = getSessionAttribute(Constants.MERCHANT_STORE, request);

		
		
		/** template **/
		StringBuilder template = new StringBuilder().append(ControllerConstants.Tiles.Customer.customer).append(".").append(store.getStoreTemplate());

		return template.toString();
		
	}
```

---

</SwmSnippet>

# Data Flow and Customer Interaction

Customer data flows from the backend to the frontend through controllers that obtain customer information from request attributes or session data. This data is then used to populate views such as dashboards, account pages, and order histories, providing a seamless and personalized user experience.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
