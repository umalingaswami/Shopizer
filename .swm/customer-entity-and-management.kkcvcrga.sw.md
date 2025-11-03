---
title: Customer Entity and Management
---
# Introduction to Customer in Shop

In the Shopizer platform, a Customer represents the end user who interacts with the online store. This entity encompasses user-specific information such as username, personal details, and order history, enabling the system to deliver a personalized shopping experience.

# Importance of Customer

The Customer entity plays a vital role in managing user sessions, securing access to personal data, and facilitating essential operations like login, registration, and account updates. It ensures that each user's data is handled uniquely and securely, maintaining the integrity of the shopping experience.

# Usage of Customer in the Codebase

Customer objects are primarily used within controller classes to process user requests. For instance, during login, the system retrieves a Customer by username to authenticate and load user data. In account management, Customer data is updated and used to render personalized views tailored to the logged-in user.

# Controllers Utilizing Customer

Multiple controllers interact with the Customer entity, including CustomerDashboardController, <SwmToken path="shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerRegistrationController.java" pos="61:4:4" line-data="public class CustomerRegistrationController extends AbstractController {">`CustomerRegistrationController`</SwmToken>, CustomerProductReviewController, <SwmToken path="shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerAccountController.java" pos="69:4:4" line-data="public class CustomerAccountController extends AbstractController {">`CustomerAccountController`</SwmToken>, CustomerLoginController, and CustomerOrdersController. Additionally, the <SwmToken path="shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerAccountController.java" pos="55:16:16" line-data="import com.salesmanager.web.shop.controller.customer.facade.CustomerFacade;">`CustomerFacade`</SwmToken> and its implementation CustomerFacadeImpl abstract service layer interactions related to Customer operations.

# <SwmToken path="shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerAccountController.java" pos="69:4:4" line-data="public class CustomerAccountController extends AbstractController {">`CustomerAccountController`</SwmToken> Overview

The <SwmToken path="shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerAccountController.java" pos="69:4:4" line-data="public class CustomerAccountController extends AbstractController {">`CustomerAccountController`</SwmToken> manages customer-specific web requests such as displaying account details and enabling password changes. It secures these actions through role-based authorization, ensuring only authenticated customers can access their account pages. The controller also retrieves the current merchant store context from the session to render store-specific templates for customer views.

# Customer Data Management

Operations involving customer data include updating personal information and changing passwords. These are facilitated through dedicated controller methods and model attributes, which handle form submissions and validation to maintain data integrity.

<SwmSnippet path="/shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerAccountController.java" line="147">

---

The platform exposes several customer-related endpoints to support account management, registration, authentication, dashboard access, product reviews, and order management. For example, the endpoint `/shop/customer/account.html` (GET) displays the customer's account page, while `/shop/customer/changePassword.html` (POST) handles password changes. These endpoints enforce role-based authorization and utilize the merchant store context to render appropriate templates.

```java
	@RequestMapping(value="/account.html", method=RequestMethod.GET)
	public String displayCustomerAccount(Model model, HttpServletRequest request, HttpServletResponse response) throws Exception {
		

	    MerchantStore store = getSessionAttribute(Constants.MERCHANT_STORE, request);

		
		
		/** template **/
		StringBuilder template = new StringBuilder().append(ControllerConstants.Tiles.Customer.customer).append(".").append(store.getStoreTemplate());

		return template.toString();
		
	}
	
	@PreAuthorize("hasRole('AUTH_CUSTOMER')")
	@RequestMapping(value="/password.html", method=RequestMethod.GET)
	public String displayCustomerChangePassword(Model model, HttpServletRequest request, HttpServletResponse response) throws Exception {
		

	    MerchantStore store = getSessionAttribute(Constants.MERCHANT_STORE, request);

		CustomerPassword customerPassword = new CustomerPassword();
		model.addAttribute("password", customerPassword);
		
		/** template **/
		StringBuilder template = new StringBuilder().append(ControllerConstants.Tiles.Customer.changePassword).append(".").append(store.getStoreTemplate());

		return template.toString();
		
	}
	
	@PreAuthorize("hasRole('AUTH_CUSTOMER')")
	@RequestMapping(value="/changePassword.html", method=RequestMethod.POST)
	public String changePassword(@Valid @ModelAttribute(value="password") CustomerPassword password, BindingResult bindingResult, Model model, HttpServletRequest request, HttpServletResponse response, Locale locale) throws Exception {
```

---

</SwmSnippet>

The <SwmPath>[shopizer/…/customer/CustomerAccountController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerAccountController.java)</SwmPath> file contains the implementation of these account management endpoints, demonstrating how customer data is retrieved, validated, and updated securely.

<SwmSnippet path="/shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerRegistrationController.java" line="100">

---

Customer registration is handled through dedicated endpoints that display the registration form and process submitted data. The endpoint `/shop/customer/registration.html` (GET) presents the registration form, while `/shop/customer/register.html` (POST) processes the registration, including validation and sending confirmation emails. These endpoints interact with the <SwmToken path="shopizer\sm-shop\src\main\java\com\salesmanager\web\shop\controller\customer\CustomerAccountController.java" pos="55:16:16" line-data="import com.salesmanager.web.shop.controller.customer.facade.CustomerFacade;">`CustomerFacade`</SwmToken> to verify existing users and register new customers.

```java
	@RequestMapping(value="/registration.html", method=RequestMethod.GET)
	public String displayRegistration(final Model model, final HttpServletRequest request, final HttpServletResponse response) throws Exception {

		MerchantStore store = (MerchantStore)request.getAttribute(Constants.MERCHANT_STORE);

		model.addAttribute( "recapatcha_public_key", coreConfiguration.getProperty( Constants.RECAPATCHA_PUBLIC_KEY ) );
		
		SecuredShopPersistableCustomer customer = new SecuredShopPersistableCustomer();
		AnonymousCustomer anonymousCustomer = (AnonymousCustomer)request.getAttribute(Constants.ANONYMOUS_CUSTOMER);
		if(anonymousCustomer!=null) {
			customer.setBilling(anonymousCustomer.getBilling());
		}
		
		model.addAttribute("customer", customer);

		/** template **/
		StringBuilder template = new StringBuilder().append(ControllerConstants.Tiles.Customer.register).append(".").append(store.getStoreTemplate());

		return template.toString();


	}

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

The <SwmPath>[shopizer/…/customer/CustomerRegistrationController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/shop/controller/customer/CustomerRegistrationController.java)</SwmPath> file implements the registration logic, showcasing how the system validates input, checks for duplicate accounts, and integrates with email services to confirm registration.

# Example: Personalizing Customer Dashboard

In the CustomerDashboardController, the Customer object is retrieved from the request attributes to customize the dashboard for the logged-in user. This demonstrates how Customer data is leveraged to personalize the user interface, enhancing the shopping experience.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
