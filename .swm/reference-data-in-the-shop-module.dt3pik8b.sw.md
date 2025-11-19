---
title: Reference Data in the Shop Module
---
# What is Reference Data in the Shop Module

Reference data in the Shop Module consists of predefined, reusable data entities that provide standardized values used throughout the e-commerce platform. These entities represent fixed lists such as countries, languages, currencies, and other common data elements essential for consistent operation.

# Purpose of Reference Data

The primary purpose of Reference data is to maintain uniformity and consistency across different parts of the application. By centralizing these common data elements, the platform ensures that all modules—such as shopping cart, checkout, and administration—use the same standardized values, which reduces discrepancies and simplifies integration.

# How Reference Data is Used

Reference entities act as a centralized source of truth. Various modules within the Shopizer platform reference these entities instead of defining their own versions of common data. This approach minimizes data duplication and streamlines data management by allowing multiple components to rely on a consistent set of values.

# Data Management and Integrity

Reference data is managed separately from transactional data to ensure its stability and integrity. This separation means that core values like country codes or currency symbols remain consistent and are not affected by day-to-day transactional changes, preserving reliable data throughout the system.

# Example Use Case

For instance, the platform uses Reference data to standardize country and currency information. This ensures that modules such as the shopping cart, checkout process, and admin panel all operate with the same country codes and currency formats, preventing inconsistencies and facilitating smoother data handling.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBU2hvcGl6ZXIlM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="Shopizer"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
