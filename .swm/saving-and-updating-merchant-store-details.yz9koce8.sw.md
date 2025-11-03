---
title: Saving and Updating Merchant Store Details
---
This document describes how administrators can update merchant store details. Submitted changes are validated and saved, and if the store code changes, a notification email is sent to the store's contact. The session and UI are updated to reflect the latest store information.

```mermaid
flowchart TD
  node1["Handling Store Save and Validation"]:::HeadingStyle
  click node1 goToHeading "Handling Store Save and Validation"
  node1 --> node2{"Store code changed?"}
  node2 -->|"Yes"| node3["Preparing and Dispatching Email"]:::HeadingStyle
  click node3 goToHeading "Preparing and Dispatching Email"
  node2 -->|"No"| node4["Updating Session and Returning the View"]:::HeadingStyle
  click node4 goToHeading "Updating Session and Returning the View"
  node3 --> node4

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Handling Store Save and Validation

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java" line="220">

---

In <SwmToken path="shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java" pos="220:5:5" line-data="	public String saveMerchantStore(@Valid @ModelAttribute(&quot;store&quot;) MerchantStore store, BindingResult result, Model model, HttpServletRequest request, HttpServletResponse response, Locale locale) throws Exception {">`saveMerchantStore`</SwmToken>, the flow starts by validating the store's identity and business date, then loads all the reference data (weights, sizes, countries, languages, currencies) needed for the admin UI. It checks if the store's <SwmPath>[shopizer/…/reference/zone/](shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/reference/zone/)</SwmPath> setup is valid, handles validation errors, and updates the store's related entities from the latest service data. This sets up everything needed before saving or triggering any side effects like email or session updates.

```java
	public String saveMerchantStore(@Valid @ModelAttribute("store") MerchantStore store, BindingResult result, Model model, HttpServletRequest request, HttpServletResponse response, Locale locale) throws Exception {
		
		setMenu(model,request);
		MerchantStore sessionStore = (MerchantStore)request.getAttribute(Constants.ADMIN_STORE);

		if(store.getId()!=null) {
			if(store.getId().intValue() != sessionStore.getId().intValue()) {
				return "redirect:/admin/store/store.html";
			}
		}
		
		Date date = new Date();
		if(!StringUtils.isBlank(store.getDateBusinessSince())) {
			try {
				date = DateUtil.getDate(store.getDateBusinessSince());
				store.setInBusinessSince(date);
			} catch (Exception e) {
				ObjectError error = new ObjectError("dateBusinessSince",messages.getMessage("message.invalid.date", locale));
				result.addError(error);
			}
		}
		
		List<Currency> currencies = currencyService.list();
		
		
		Language language = (Language)request.getAttribute("LANGUAGE");
		List<Language> languages = languageService.getLanguages();
		
		//get countries
		List<Country> countries = countryService.getCountries(language);
		
		List<Weight> weights = new ArrayList<Weight>();
		weights.add(new Weight("LB",messages.getMessage("label.generic.weightunit.LB", locale)));
		weights.add(new Weight("KG",messages.getMessage("label.generic.weightunit.KG", locale)));
		
		List<Size> sizes = new ArrayList<Size>();
		sizes.add(new Size("CM",messages.getMessage("label.generic.sizeunit.CM", locale)));
		sizes.add(new Size("IN",messages.getMessage("label.generic.sizeunit.IN", locale)));
		
		model.addAttribute("weights",weights);
		model.addAttribute("sizes",sizes);
		
		model.addAttribute("countries", countries);
		model.addAttribute("languages",languages);
		model.addAttribute("currencies",currencies);
		
		
		Country c = store.getCountry();
		List<Zone> zonesList = zoneService.getZones(c, language);
		
		if((zonesList==null || zonesList.size()==0) && StringUtils.isBlank(store.getStorestateprovince())) {
			
			ObjectError error = new ObjectError("zone.code",messages.getMessage("merchant.zone.invalid", locale));
			result.addError(error);
			
		}

		if (result.hasErrors()) {
			return "admin-store";
		}
		
		//get country
		Country country = store.getCountry();
		country = countryService.getByCode(country.getIsoCode());
		Zone zone = store.getZone();
		if(zone!=null) {
			zone = zoneService.getByCode(zone.getCode());
		}
		Currency currency = store.getCurrency();
		currency = currencyService.getById(currency.getId());

		List<Language> supportedLanguages = store.getLanguages();
		List<Language> supportedLanguagesList = new ArrayList<Language>();
		Map<String,Language> languagesMap = languageService.getLanguagesMap();
		for(Language lang : supportedLanguages) {
			
			Language l = languagesMap.get(lang.getCode());
			if(l!=null) {
				supportedLanguagesList.add(l);
			}
			
		}
```

---

</SwmSnippet>

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java" line="303">

---

Here, after saving the store, we check if the store code changed. If so, we build a localized email using template tokens and send it to the store's email address. This notifies admins about the new or updated store. The email content and tokens are built using repository-specific services and localization.

```java
		Language defaultLanguage = store.getDefaultLanguage();
		defaultLanguage = languageService.getById(defaultLanguage.getId());
		if(defaultLanguage!=null) {
			store.setDefaultLanguage(defaultLanguage);
		}
		
		Locale storeLocale = LocaleUtils.getLocale(defaultLanguage);
		
		store.setStoreTemplate(sessionStore.getStoreTemplate());
		store.setCountry(country);
		store.setZone(zone);
		store.setCurrency(currency);
		store.setDefaultLanguage(defaultLanguage);
		store.setLanguages(supportedLanguagesList);
		store.setLanguages(supportedLanguagesList);

		
		merchantStoreService.saveOrUpdate(store);
		
		if(!store.getCode().equals(sessionStore.getCode())) {//create store
			//send email
			
			try {


				Map<String, String> templateTokens = EmailUtils.createEmailObjectsMap(request.getContextPath(), store, messages, storeLocale);
				templateTokens.put(EmailConstants.EMAIL_NEW_STORE_TEXT, messages.getMessage("email.newstore.text", storeLocale));
				templateTokens.put(EmailConstants.EMAIL_STORE_NAME, messages.getMessage("email.newstore.name",new String[]{store.getStorename()},storeLocale));
				templateTokens.put(EmailConstants.EMAIL_ADMIN_STORE_INFO_LABEL, messages.getMessage("email.newstore.info",storeLocale));

				templateTokens.put(EmailConstants.EMAIL_ADMIN_URL_LABEL, messages.getMessage("label.adminurl",storeLocale));
				templateTokens.put(EmailConstants.EMAIL_ADMIN_URL, FilePathUtils.buildAdminUri(store, request));
	
				
				Email email = new Email();
				email.setFrom(store.getStorename());
				email.setFromEmail(store.getStoreEmailAddress());
				email.setSubject(messages.getMessage("email.newstore.title",storeLocale));
				email.setTo(store.getStoreEmailAddress());
				email.setTemplateName(NEW_STORE_TMPL);
				email.setTemplateTokens(templateTokens);
	
	
				
				emailService.sendHtmlEmail(store, email);
			
			} catch (Exception e) {
				LOGGER.error("Cannot send email to user",e);
			}
			
		}

```

---

</SwmSnippet>

## Preparing and Dispatching Email

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Given a merchant store and an email"]
  click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:25:31"
  node1 --> node2["Retrieve email configuration for the merchant store"]
  click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:27:27"
  node2 --> node3["Set sender with the store's email configuration"]
  click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:29:29"
  node3 --> node4["Send HTML email to the recipient"]
  click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:30:30"
  node4 --> node5["Email sent"]
  click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java:30:31"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Given a merchant store and an email"]
%%   click node1 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:25:31"
%%   node1 --> node2["Retrieve email configuration for the merchant store"]
%%   click node2 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:27:27"
%%   node2 --> node3["Set sender with the store's email configuration"]
%%   click node3 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:29:29"
%%   node3 --> node4["Send HTML email to the recipient"]
%%   click node4 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:30:30"
%%   node4 --> node5["Email sent"]
%%   click node5 openCode "<SwmPath>[shopizer/…/service/EmailServiceImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java)</SwmPath>:30:31"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/business/system/service/EmailServiceImpl.java" line="25">

---

This part loads the store's email config and passes the email to the sender for delivery.

```java
	public void sendHtmlEmail(MerchantStore store, Email email) throws ServiceException, Exception {

		EmailConfig emailConfig = getEmailConfiguration(store);
		
		sender.setEmailConfig(emailConfig);
		sender.send(email);
	}
```

---

</SwmSnippet>

## Composing and Sending the Email Message

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java" line="40">

---

This part builds the email content (text and HTML), configures SMTP, and preps the message, then moves to order processing if needed.

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

### Order-Related Email Processing

See <SwmLink doc-title="Order Processing Flow">[Order Processing Flow](.swm%5Corder-processing-flow.42xu5bqj.sw.md)</SwmLink>

### Finalizing and Sending the Email

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Prepare HTML email content (HTML format)"]
    click node1 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java:129:146"
    node1 --> node2{"Are there attachments?"}
    click node2 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java:159:163"
    node2 -->|"No"| node3["Add HTML body part to email"]
    click node3 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java:152:155"
    node3 --> node4["Send email to recipient"]
    click node4 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java:168:169"
    node2 -.->|"Yes (inactive)"| node5["Add attachment to email (inactive)"]
    click node5 openCode "shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java:159:163"
    node5 -.-> node3
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Prepare HTML email content (HTML format)"]
%%     click node1 openCode "<SwmPath>[shopizer/…/email/HtmlEmailSenderImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java)</SwmPath>:129:146"
%%     node1 --> node2{"Are there attachments?"}
%%     click node2 openCode "<SwmPath>[shopizer/…/email/HtmlEmailSenderImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java)</SwmPath>:159:163"
%%     node2 -->|"No"| node3["Add HTML body part to email"]
%%     click node3 openCode "<SwmPath>[shopizer/…/email/HtmlEmailSenderImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java)</SwmPath>:152:155"
%%     node3 --> node4["Send email to recipient"]
%%     click node4 openCode "<SwmPath>[shopizer/…/email/HtmlEmailSenderImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java)</SwmPath>:168:169"
%%     node2 -.->|"Yes (inactive)"| node5["Add attachment to email (inactive)"]
%%     click node5 openCode "<SwmPath>[shopizer/…/email/HtmlEmailSenderImpl.java](shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java)</SwmPath>:159:163"
%%     node5 -.-> node3
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-core/src/main/java/com/salesmanager/core/modules/email/HtmlEmailSenderImpl.java" line="129">

---

After order processing, we finish building the email and send it.

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

## Updating Session and Returning the View

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Get latest merchant store data using store code"]
    click node1 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java:355:355"
    node1 --> node2["Update session with refreshed store"]
    click node2 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java:359:359"
    node2 --> node3["Set 'success' status in model"]
    click node3 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java:362:362"
    node3 --> node4["Set store data in model"]
    click node4 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java:363:363"
    node4 --> node5["Show admin store page"]
    click node5 openCode "shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java:366:366"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Get latest merchant store data using store code"]
%%     click node1 openCode "<SwmPath>[shopizer/…/merchant/MerchantStoreController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java)</SwmPath>:355:355"
%%     node1 --> node2["Update session with refreshed store"]
%%     click node2 openCode "<SwmPath>[shopizer/…/merchant/MerchantStoreController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java)</SwmPath>:359:359"
%%     node2 --> node3["Set 'success' status in model"]
%%     click node3 openCode "<SwmPath>[shopizer/…/merchant/MerchantStoreController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java)</SwmPath>:362:362"
%%     node3 --> node4["Set store data in model"]
%%     click node4 openCode "<SwmPath>[shopizer/…/merchant/MerchantStoreController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java)</SwmPath>:363:363"
%%     node4 --> node5["Show admin store page"]
%%     click node5 openCode "<SwmPath>[shopizer/…/merchant/MerchantStoreController.java](shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java)</SwmPath>:366:366"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/shopizer/sm-shop/src/main/java/com/salesmanager/web/admin/controller/merchant/MerchantStoreController.java" line="355">

---

Back in `MerchantStoreController.saveMerchantStore`, after sending the email, we reload the store from the database, update the session with the latest store data, set a success flag, and return the admin store view. This keeps the session and UI in sync with the latest changes.

```java
		sessionStore = merchantStoreService.getMerchantStore(sessionStore.getCode());
		
		
		//update session store
		request.getSession().setAttribute(Constants.ADMIN_STORE, sessionStore);


		model.addAttribute("success","success");
		model.addAttribute("store", store);

		
		return "admin-store";
	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
