---
title: User Core Model Overview
---
# Overview of User Core Model

The User core model represents an individual who interacts with the system. It encapsulates essential information and attributes required to manage user accounts effectively.

This model holds critical data such as identification details, authentication credentials, and personal information. These include username, password, email, first and last names, and security questions, which are fundamental for user management.

Beyond basic data, the User model tracks metadata like the last access time and login time, which are useful for monitoring user activity and enhancing security.

The User model serves as the foundation for associating permissions, roles, and groups. This association enables the system to control access and define the capabilities of each user within the application.

Designed with extensibility in mind, the User model allows additional properties or relationships to be added as needed. This flexibility supports various business requirements and evolving system needs.

# Purpose and Usage of the User Model

The primary purpose of the User class is to manage user accounts, including authentication and authorization processes. It links users to groups and merchant stores, which helps in defining their permissions and roles within the system.

To use the User model, developers instantiate it with required credentials such as username, password, and email. Additional properties like first name, last name, and default language can be set to enrich the user profile.

Users can be assigned to groups, which facilitates managing their access rights and permissions efficiently. This grouping mechanism is central to the system's role-based access control.

<SwmSnippet path="/shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" line="55">

---

The constructor <SwmToken path="shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" pos="55:3:3" line-data="	public User(String userName,String password, String email) {">`User`</SwmToken>`(`<SwmToken path="shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" pos="55:5:5" line-data="	public User(String userName,String password, String email) {">`String`</SwmToken>` `<SwmToken path="shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" pos="55:7:7" line-data="	public User(String userName,String password, String email) {">`userName`</SwmToken>`, `<SwmToken path="shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" pos="55:5:5" line-data="	public User(String userName,String password, String email) {">`String`</SwmToken>` `<SwmToken path="shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" pos="55:11:11" line-data="	public User(String userName,String password, String email) {">`password`</SwmToken>`, `<SwmToken path="shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" pos="55:5:5" line-data="	public User(String userName,String password, String email) {">`String`</SwmToken>` `<SwmToken path="shopizer/sm-core-model/src/main/java/com/salesmanager/core/business/user/model/User.java" pos="55:16:16" line-data="	public User(String userName,String password, String email) {">`email`</SwmToken>`)` initializes a new user instance with the provided credentials. It sets the username, password, and email fields accordingly, ensuring that essential authentication data is captured at creation.

```java
	public User(String userName,String password, String email) {
		
		this.adminName = userName;
		this.adminPassword = password;
		this.adminEmail = email;
	}
```

---

</SwmSnippet>

This snippet shows the constructor implementation that initializes the core fields of the User object.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
