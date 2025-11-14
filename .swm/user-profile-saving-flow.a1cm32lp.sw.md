---
title: User profile saving flow
---
This document explains the process of saving user profiles in the admin interface. It ensures that user group assignments are validated and preserved correctly, passwords are securely handled for new users, and notification emails are sent when new users are created. The flow receives user data as input and outputs a saved user profile with confirmation feedback.

```mermaid
flowchart TD
  node1["Managing user save with validation, groups, password encoding, and notification
(Managing user save with validation, groups, password, and notification)"]:::HeadingStyle
  node2{"Is user new?
(Managing user save with validation, groups, password, and notification)"}:::HeadingStyle
  node3["Save user profile
(Managing user save with validation, groups, password, and notification)"]:::HeadingStyle
  node4["Configuring and delegating email sending"]:::HeadingStyle
  node5["Completing user save with success feedback"]:::HeadingStyle

  node1 --> node2
  node2 -- Yes --> node3
  node2 -- No --> node3
  node3 -- New user --> node4
  node3 --> node5
  node4 --> node5

  click node1 goToHeading "Managing user save with validation, groups, password, and notification"
  click node2 goToHeading "Managing user save with validation, groups, password, and notification"
  click node3 goToHeading "Managing user save with validation, groups, password, and notification"
  click node4 goToHeading "Configuring and delegating email sending"
  click node5 goToHeading "Completing user save with success feedback"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Managing user save with validation, groups, password, and notification

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Prepare user data and assign groups"]
    subgraph loop1["For each submitted user group"]
        node1 --> node1
    end
    node1 --> node2["Configuring and delegating email sending"]
    node2 --> node3{"Is new user?"}
    node3 -->|"Yes"| node4["Send notification email"]
    node3 -->|"No"| node4
    node4 --> node5["Return success view"]

    click node1 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java:472:506"
    
    click node3 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java:585:629"
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:25:50"
    click node5 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java:634:636"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node2 goToHeading "Configuring and delegating email sending"
node2:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Prepare user data and assign groups"]
%%     subgraph loop1["For each submitted user group"]
%%         node1 --> node1
%%     end
%%     node1 --> node2["Configuring and delegating email sending"]
%%     node2 --> node3{"Is new user?"}
%%     node3 -->|"Yes"| node4["Send notification email"]
%%     node3 -->|"No"| node4
%%     node4 --> node5["Return success view"]
%% 
%%     click node1 openCode "<SwmPath>[shopizer/…/user/UserController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java)</SwmPath>:472:506"
%%     
%%     click node3 openCode "<SwmPath>[shopizer/…/user/UserController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java)</SwmPath>:585:629"
%%     click node4 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:25:50"
%%     click node5 openCode "<SwmPath>[shopizer/…/user/UserController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java)</SwmPath>:634:636"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node2 goToHeading "Configuring and delegating email sending"
%% node2:::HeadingStyle
```

This section manages saving user data with validation, group assignments, password handling, and notification email sending.

| Category        | Rule Name                                    | Description                                                                                                                                              |
| --------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Data validation | Validate User Group Assignments              | User group assignments must be validated to ensure only authorized groups are assigned, especially handling the SUPERADMIN group carefully.              |
| Data validation | Handle Validation Errors                     | If validation errors occur during user data submission, the system must return the user profile view with error messages instead of saving.              |
| Business logic  | Preserve Superadmin Group                    | If the user being edited currently has the SUPERADMIN group, this group cannot be removed from the user.                                                 |
| Business logic  | Password Encoding for New Users              | When creating a new user, the password must be encoded before saving to ensure security. For existing users, the original encoded password is preserved. |
| Business logic  | Send Notification Email on New User Creation | When a new user is created, an email notification with account details is sent to the user's email address.                                              |
| Business logic  | Return Success View After Save               | After successfully saving the user data and sending any necessary notifications, the system returns a success view to the administrator.                 |

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" line="472">

---

The start of <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="472:5:5" line-data="	public String saveUser(@Valid @ModelAttribute(&quot;user&quot;) User user, BindingResult result, Model model, HttpServletRequest request, Locale locale) throws Exception {">`saveUser`</SwmToken> sets up user context, validates groups especially SUPERADMIN, and prepares for password handling and email notification.

```java
	public String saveUser(@Valid @ModelAttribute("user") User user, BindingResult result, Model model, HttpServletRequest request, Locale locale) throws Exception {


		setMenu(model,request);
		
		MerchantStore store = (MerchantStore)request.getAttribute(Constants.ADMIN_STORE);

		
		this.populateUserObjects(user, store, model, locale);
		
		Language language = user.getDefaultLanguage();
		
		Language l = languageService.getById(language.getId());
		
		user.setDefaultLanguage(l);
		
		Locale userLocale = LocaleUtils.getLocale(l);
		
		
		
		User dbUser = null;
		
		//edit mode, need to get original user important information
		if(user.getId()!=null) {
			dbUser = userService.getByUserName(user.getAdminName());
			if(dbUser==null) {
				return "redirect://admin/users/displayUser.html";
			}
		}

		List<Group> submitedGroups = user.getGroups();
		Set<Integer> ids = new HashSet<Integer>();
		for(Group group : submitedGroups) {
			ids.add(Integer.parseInt(group.getGroupName()));
		}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" line="537">

---

Next in <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="472:5:5" line-data="	public String saveUser(@Valid @ModelAttribute(&quot;user&quot;) User user, BindingResult result, Model model, HttpServletRequest request, Locale locale) throws Exception {">`saveUser`</SwmToken>, the code checks if the user is editing their own record and scans existing groups to preserve SUPERADMIN if present, preventing its removal.

```java
		Group superAdmin = null;
		
		if(user.getId()!=null && user.getId()>0) {
			if(user.getId().longValue()!=dbUser.getId().longValue()) {
				return "redirect://admin/users/displayUser.html";
			}
			
			List<Group> groups = dbUser.getGroups();
			//boolean removeSuperAdmin = true;
			for(Group group : groups) {
				//can't revoke super admin
				if(group.getGroupName().equals("SUPERADMIN")) {
					superAdmin = group;
				}
			}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" line="562">

---

Here, the function finalizes group assignments, encodes passwords conditionally, saves the user, and if new, builds and sends an email notification using <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java" pos="16:4:4" line-data="public class EmailServiceImpl implements EmailService {">`EmailServiceImpl`</SwmToken> to inform the user about their account.

```java
		if(superAdmin!=null) {
			ids.add(superAdmin.getId());
		}

		
		List<Group> newGroups = groupService.listGroupByIds(ids);

		//set actual user groups
		user.setGroups(newGroups);
		
		if (result.hasErrors()) {
			return ControllerConstants.Tiles.User.profile;
		}
		
		String decodedPassword = user.getAdminPassword();
		if(user.getId()!=null && user.getId()>0) {
			user.setAdminPassword(dbUser.getAdminPassword());
		} else {
			String encoded = passwordEncoder.encodePassword(user.getAdminPassword(),null);
			user.setAdminPassword(encoded);
		}
		
		
		if(user.getId()==null || user.getId().longValue()==0) {
			
			//save or update user
			userService.saveOrUpdate(user);
			
			try {

				//creation of a user, send an email
				String userName = user.getFirstName();
				if(StringUtils.isBlank(userName)) {
					userName = user.getAdminName();
				}
				String[] userNameArg = {userName};
				
				
				Map<String, String> templateTokens = EmailUtils.createEmailObjectsMap(request.getContextPath(), store, messages, userLocale);
				templateTokens.put(EmailConstants.EMAIL_NEW_USER_TEXT, messages.getMessage("email.greeting", userNameArg, userLocale));
				templateTokens.put(EmailConstants.EMAIL_USER_FIRSTNAME, user.getFirstName());
				templateTokens.put(EmailConstants.EMAIL_USER_LASTNAME, user.getLastName());
				templateTokens.put(EmailConstants.EMAIL_ADMIN_USERNAME_LABEL, messages.getMessage("label.generic.username",userLocale));
				templateTokens.put(EmailConstants.EMAIL_ADMIN_NAME, user.getAdminName());
				templateTokens.put(EmailConstants.EMAIL_TEXT_NEW_USER_CREATED, messages.getMessage("email.newuser.text",userLocale));
				templateTokens.put(EmailConstants.EMAIL_ADMIN_PASSWORD_LABEL, messages.getMessage("label.generic.password",userLocale));
				templateTokens.put(EmailConstants.EMAIL_ADMIN_PASSWORD, decodedPassword);
				templateTokens.put(EmailConstants.EMAIL_ADMIN_URL_LABEL, messages.getMessage("label.adminurl",userLocale));
				templateTokens.put(EmailConstants.EMAIL_ADMIN_URL, FilePathUtils.buildAdminUri(store, request));
	
				
				Email email = new Email();
				email.setFrom(store.getStorename());
				email.setFromEmail(store.getStoreEmailAddress());
				email.setSubject(messages.getMessage("email.newuser.title",userLocale));
				email.setTo(user.getAdminEmail());
				email.setTemplateName(NEW_USER_TMPL);
				email.setTemplateTokens(templateTokens);
	
	
				
				emailService.sendHtmlEmail(store, email);
			
			} catch (Exception e) {
				LOGGER.error("Cannot send email to user",e);
			}
			
		} else {
			//save or update user
			userService.saveOrUpdate(user);
		}

```

---

</SwmSnippet>

## Configuring and delegating email sending

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start sending HTML email for MerchantStore"]
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:25:31"
    node1 --> node2["Retrieve email configuration for MerchantStore"]
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:27:27"
    node2 --> node3["Set email configuration on sender"]
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:29:29"
    node3 --> node4["Send HTML email to recipient"]
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:30:30"
    node4 --> node5["Email sent successfully"]
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:31:31"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start sending HTML email for <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="477:1:1" line-data="		MerchantStore store = (MerchantStore)request.getAttribute(Constants.ADMIN_STORE);">`MerchantStore`</SwmToken>"]
%%     click node1 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:25:31"
%%     node1 --> node2["Retrieve email configuration for <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="477:1:1" line-data="		MerchantStore store = (MerchantStore)request.getAttribute(Constants.ADMIN_STORE);">`MerchantStore`</SwmToken>"]
%%     click node2 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:27:27"
%%     node2 --> node3["Set email configuration on sender"]
%%     click node3 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:29:29"
%%     node3 --> node4["Send HTML email to recipient"]
%%     click node4 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:30:30"
%%     node4 --> node5["Email sent successfully"]
%%     click node5 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:31:31"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

This section describes the process of configuring and delegating the sending of HTML emails for a <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="477:1:1" line-data="		MerchantStore store = (MerchantStore)request.getAttribute(Constants.ADMIN_STORE);">`MerchantStore`</SwmToken> in the Shopizer platform.

| Category        | Rule Name                          | Description                                                                                                                                                                                                                                                                                                                                                                    |
| --------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Data validation | Valid email configuration required | Each <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="477:1:1" line-data="		MerchantStore store = (MerchantStore)request.getAttribute(Constants.ADMIN_STORE);">`MerchantStore`</SwmToken> must have a valid email configuration before sending any emails.                                                    |
| Business logic  | Store-specific email configuration | The email must be sent using the email configuration specific to the <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="477:1:1" line-data="		MerchantStore store = (MerchantStore)request.getAttribute(Constants.ADMIN_STORE);">`MerchantStore`</SwmToken> to ensure correct sender details and SMTP settings. |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java" line="25">

---

SendHtmlEmail sets up email configuration for the store and then calls HtmlEmailSenderImpl.send to handle the actual email sending process.

```java
	public void sendHtmlEmail(MerchantStore store, Email email) throws ServiceException, Exception {

		EmailConfig emailConfig = getEmailConfiguration(store);
		
		sender.setEmailConfig(emailConfig);
		sender.send(email);
	}
```

---

</SwmSnippet>

## Preparing multipart email content and SMTP setup

This section handles the preparation of multipart email content and SMTP setup for sending emails within the Shopizer platform.

| Category        | Rule Name                             | Description                                                                                                                          |
| --------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Data validation | Email sender and recipient validation | The email must have a valid sender address and recipient address before sending.                                                     |
| Business logic  | Generate multipart email content      | The email content must be generated as multipart with both plain text and HTML parts using templates and tokens for personalization. |

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java" line="40">

---

In send, the function sets up the email headers, loads and processes templates for both plain text and HTML parts, applies SMTP settings if available, and prepares the multipart email content. It then calls OrderServiceImpl.process before finishing.

```java
	public void send(Email email)
			throws Exception {
		
		final String eml = email.getFrom();
		final String from = email.getFromEmail();
		final String to = email.getTo();
		final String subject = email.getSubject();
		final String tmpl = email.getTemplateName();
		final Map<String,String> templateTokens = email.getTemplateTokens();

		MimeMessagePreparator preparator = new MimeMessagePreparator() {
			public void prepare(MimeMessage mimeMessage)
					throws MessagingException, IOException {
				
				JavaMailSenderImpl impl = (JavaMailSenderImpl)mailSender;
				// if email configuration is present in Database, use the same
				if(emailConfig != null) {
					impl.setProtocol(emailConfig.getProtocol());
					impl.setHost(emailConfig.getHost());
					impl.setPort(Integer.parseInt(emailConfig.getPort()));
					impl.setUsername(emailConfig.getUsername());
					impl.setPassword(emailConfig.getPassword());
					
					Properties prop = new Properties();
					prop.put("mail.smtp.auth", emailConfig.isSmtpAuth());
					prop.put("mail.smtp.starttls.enable", emailConfig.isStarttls());
					impl.setJavaMailProperties(prop);
				}
				
				mimeMessage.setRecipient(Message.RecipientType.TO, new InternetAddress(to));

				InternetAddress inetAddress = new InternetAddress();

				inetAddress.setPersonal(eml);
				inetAddress.setAddress(from);

				mimeMessage.setFrom(inetAddress);
				mimeMessage.setSubject(subject);

				Multipart mp = new MimeMultipart("alternative");

				// Create a "text" Multipart message
				BodyPart textPart = new MimeBodyPart();
				freemarkerMailConfiguration.setClassForTemplateLoading(HtmlEmailSenderImpl.class, "/");
				Template textTemplate = freemarkerMailConfiguration.getTemplate(new StringBuilder(TEMPLATE_PATH).append("").append("/").append(tmpl).toString());
				final StringWriter textWriter = new StringWriter();
				try {
					textTemplate.process(templateTokens, textWriter);
				} catch (TemplateException e) {
					throw new MailPreparationException(
							"Can't generate text mail", e);
				}
				textPart.setDataHandler(new javax.activation.DataHandler(
						new javax.activation.DataSource() {
							public InputStream getInputStream()
									throws IOException {
								//return new StringBufferInputStream(textWriter
								//		.toString());
								return new ByteArrayInputStream(textWriter
										.toString().getBytes(CHARSET));
							}

							public OutputStream getOutputStream()
									throws IOException {
								throw new IOException("Read-only data");
							}

							public String getContentType() {
								return "text/plain";
							}

							public String getName() {
								return "main";
							}
						}));
				mp.addBodyPart(textPart);

				// Create a "HTML" Multipart message
				Multipart htmlContent = new MimeMultipart("related");
				BodyPart htmlPage = new MimeBodyPart();
				freemarkerMailConfiguration.setClassForTemplateLoading(HtmlEmailSenderImpl.class, "/");
				Template htmlTemplate = freemarkerMailConfiguration.getTemplate(new StringBuilder(TEMPLATE_PATH).append("").append("/").append(tmpl).toString());
				final StringWriter htmlWriter = new StringWriter();
				try {
					htmlTemplate.process(templateTokens, htmlWriter);
				} catch (TemplateException e) {
					throw new MailPreparationException(
							"Can't generate HTML mail", e);
				}
```

---

</SwmSnippet>

### Order processing integration in email sending

This section covers the integration of order processing with the email sending functionality, ensuring that order-related emails are sent appropriately.

| Category       | Rule Name                    | Description                                                                                                          |
| -------------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Business logic | Immediate order confirmation | Order confirmation emails must be sent immediately after an order is successfully processed.                         |
| Business logic | Order details in emails      | Emails related to order processing must include key order details such as order ID, customer name, and order status. |

See <SwmLink doc-title="Order processing flow">[Order processing flow](.swm%5Corder-processing-flow.xiujz461.sw.md)</SwmLink>

### Finalizing and sending the email message

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java" line="129">

---

We just returned from OrderServiceImpl.process, and now in HtmlEmailSenderImpl.send, the function finishes assembling the multipart email, sets it on the message, and sends it using <SwmToken path="shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java" pos="168:1:1" line-data="		mailSender.send(preparator);">`mailSender`</SwmToken>.

```java
				htmlPage.setDataHandler(new javax.activation.DataHandler(
						new javax.activation.DataSource() {
							public InputStream getInputStream()
									throws IOException {
								//return new StringBufferInputStream(htmlWriter
								//		.toString());
								return new ByteArrayInputStream(textWriter
										.toString().getBytes(CHARSET));
							}

							public OutputStream getOutputStream()
									throws IOException {
								throw new IOException("Read-only data");
							}

							public String getContentType() {
								return "text/html";
							}

							public String getName() {
								return "main";
							}
						}));
				htmlContent.addBodyPart(htmlPage);
				BodyPart htmlPart = new MimeBodyPart();
				htmlPart.setContent(htmlContent);
				mp.addBodyPart(htmlPart);

				mimeMessage.setContent(mp);

				// if(attachment!=null) {
				// MimeMessageHelper messageHelper = new
				// MimeMessageHelper(mimeMessage, true);
				// messageHelper.addAttachment(attachmentFileName, attachment);
				// }

			}
		};

		mailSender.send(preparator);
	}
```

---

</SwmSnippet>

## Completing user save with success feedback

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" line="634">

---

We just returned from EmailServiceImpl.sendHtmlEmail, and now in <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/user/UserController.java" pos="472:5:5" line-data="	public String saveUser(@Valid @ModelAttribute(&quot;user&quot;) User user, BindingResult result, Model model, HttpServletRequest request, Locale locale) throws Exception {">`saveUser`</SwmToken>, the function adds a success flag to the model and returns the profile view to finish the flow.

```java
		model.addAttribute("success","success");
		return ControllerConstants.Tiles.User.profile;
	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
