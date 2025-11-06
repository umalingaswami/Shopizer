---
title: Customer Populator Overview
---
# What is Customer Populator

Customer Populator refers to a set of classes within the Populator package that handle the transformation and mapping of customer data between different representations used throughout the system.

At its core, the Customer entity represents a user or buyer in the system, encapsulating personal details, addresses, preferences, and other relevant customer information.

The Customer Populator classes facilitate converting customer data from persistable input forms, such as data received from client requests, into the core Customer model used in business logic and persistence layers.

For example, the CustomerPopulator class is responsible for creating and populating a Customer entity from a persistable customer input, preparing it for saving or further processing within the system.

Complementing this, the PersistableCustomerPopulator converts data from a persistable customer representation into the Customer model, while the ReadableCustomerPopulator transforms a Customer model into a ReadableCustomer format suitable for presentation or API responses.

Specialized populators such as PersistableCustomerBillingAddressPopulator and PersistableCustomerShippingAddressPopulator focus on mapping billing and shipping address data into the Customer model, ensuring accurate and complete address information.

Similarly, CustomerEntityPopulator assists in converting between the Customer model and CustomerEntity, which may represent the database layer or another system representation.

Additional populators like CustomerDeliveryAddressPopulator and CustomerBillingAddressPopulator extract address information from the Customer model into address objects, supporting address management and separation of concerns.

# Purpose and Usage

The primary purpose of Customer Populator classes is to centralize and standardize the transformation of customer data across different layers of the application, including persistence, business logic, and presentation.

By using these populators, the system ensures consistency and correctness when handling customer data, whether it is being received from client input, stored in the database, or prepared for API responses.

This approach also promotes separation of concerns, as each populator class focuses on a specific transformation task, such as handling billing addresses or converting to a readable format.

# Example of Customer Populator in Action

An example is the ReadableCustomerPopulator, which takes a Customer domain model and converts it into a ReadableCustomer object. This object is tailored for use in API responses or user interface displays, containing only the necessary and formatted customer information.

This conversion process involves mapping fields from the Customer model to the ReadableCustomer, potentially formatting data or excluding sensitive information.

Similarly, the PersistableCustomerPopulator takes input data representing a customer, such as from a web form, and populates a Customer model instance that can be used for business logic or persisted to the database.

# Summary

Customer Populator classes are essential components that manage the transformation of customer data between various representations within the system. They ensure data integrity, consistency, and proper separation of concerns across different application layers.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
