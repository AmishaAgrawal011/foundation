# Shipping Gateway Application: Project Requirements, Data Model, and API Specification

## Project Overview

This project aims to build a lightweight shipping gateway application that simplifies shipping management for e-commerce businesses. The application will provide a unified API for integrating with multiple shipping providers (e.g., FedEx, UPS, USPS) and offer a user-friendly interface for customers to manage their shipping needs. The application will not persist shipment data; all shipment details will be passed within the API requests and responses.

## Key Requirements

1.  **Customer/User Management:**
    *   Securely handle customer registration and account management.
    *   Provide an interface for managing API keys and authentication.
    *   Ensure data privacy and security.

2.  **Shipping API Integration and Abstraction:**
    *   Seamlessly integrate with multiple shipping providers (e.g., FedEx, UPS, USPS).
    *   Abstract the complexities of each provider's API into a simplified, unified API for customers.
    *   Enable label generation, tracking, and other essential shipping functions through the API.
    *   Ensure reliable communication and error handling with shipping providers.

## Proposed Solutions

### Technology Stack

*   **Moqui Framework:** For building the web interface, backend logic, and API facade.
*   **Apache Shiro:** For user authentication, session management, and API key-based authentication.
*   **Database (MySQL):** For storing customer data, API keys, and shipping information.
*   **SDKs/Libraries:** Utilize official SDKs or libraries from shipping providers (if available).

## Data Model and Entity Relationships

### Party Roles

The application recognizes two primary party roles:

1.  **Customers:** Organizations that utilize the shipping gateway to manage their shipping operations.
2.  **Carriers:** Shipping providers (e.g., FedEx, UPS, DHL) that integrate with the application.

### Entities

#### 1. [Party](../../udm/beginner/party.md)
*   **Purpose:** Represents both customers and carriers as organizations.

### Shipping Gateway Configuration Entities

Moqui Mantle uses a set of interconnected entities to manage shipping gateway configurations:

#### 1. [ShippingGatewayConfig](ShippingGatewayConfig.md)

#### 2. [ShippingGatewayBoxType](ShippingGatewayBoxType.md)

#### 3. [ShippingGatewayCarrier](ShippingGatewayCarrier.md)

#### 4. [ShippingGatewayMethod](ShippingGatewayMethod.md)

## Shipping Gateway Interfaces

These interfaces define the contract that your shipping gateway integrations will need to fulfill to interact with the Moqui Framework and provide shipping functionality:

1.  **`get#OrderShippingRate`:**
    *   **Purpose:** Calculates the shipping rate for an entire order, potentially consisting of multiple items and packages.
    *   **Key Inputs:** `orderId`, `orderPartSeqId`, `shippingGatewayConfigId`, package information, etc.
    *   **Key Outputs:** `shippingTotal`, `orderItemSeqId` (if an order item is created for the shipping charge).

2.  **`get#ShippingRatesBulk`:**
    *   **Purpose:** Retrieves shipping rates in bulk for multiple carrier services and packages.
    *   **Key Inputs:** `shippingGatewayConfigId`, list of carrier shipment methods, origin and destination details, package information.
    *   **Key Outputs:** List of `shippingRateInfoList` (containing rate details for each carrier/method combination), potentially updated origin/destination addresses.

3.  **`get#AutoPackageInfo`:**
    *   **Purpose:** Automatically determines package information (e.g., box type, weight) based on the items in the order.
    *   **Key Inputs:** List of `itemInfoList` (containing product IDs and quantities).
    *   **Key Outputs:** List of `packageInfoList` (containing package details).

4.  **`get#ShippingRate`:**
    *   **Purpose:** Calculates the shipping rate for a single shipment package and route segment.
    *   **Key Inputs:** `shipmentId`, `shipmentRouteSegmentSeqId`, `shipmentPackageSeqId`, `shippingGatewayConfigId`.

5.  **`request#ShippingLabels`:**
    *   **Purpose:** Requests shipping labels from the carrier for a shipment (single package or all packages).
    *   **Key Inputs:** `shipmentId`, `shipmentRouteSegmentSeqId`, `shipmentPackageSeqId` (optional), `shippingGatewayConfigId`.

6.  **`refund#ShippingLabels`:**
    *   **Purpose:** Initiates a refund for shipping labels.
    *   **Key Inputs:** `shipmentId`, `shipmentRouteSegmentSeqId`, `shipmentPackageSeqId` (optional), `shippingGatewayConfigId`.

7.  **`track#ShippingLabels`:**
    *   **Purpose:** Tracks the status of shipments.
    *   **Key Inputs:** `shipmentId`, `shipmentRouteSegmentSeqId`, `shipmentPackageSeqId` (optional), `shippingGatewayConfigId`.

8.  **`validate#ShippingPostalAddress`:**
    *   **Purpose:** Validates a shipping address using the specified gateway configuration.
    *   **Key Inputs:** `contactMechId`, `partyId`, `facilityId`, `shippingGatewayConfigId`.
    *   **Key Outputs:** Potentially updated `contactMechId` if the address is corrected.

**Key Services for Building Integrations**

*   **`calculate#OrderShipping`:** This service is a high-level entry point for calculating shipping costs for an entire order. It calls the appropriate `getOrderRateServiceName` based on the shipping gateway configuration.

*   **`calculate#OrderPartShipping`:** This service calculates shipping for a specific part of an order. It handles tasks like deleting existing shipping charges, determining package information (if the `getAutoPackageInfoName` service is available), and calling the `getOrderRateServiceName` to get the actual rate.

*   **`get#OrderShippingRatesBulk`:** This service retrieves shipping rates in bulk for multiple carriers and methods. It interacts with both the `getShippingRatesBulkName` and `getOrderRateServiceName` services based on the gateway configuration.

*   **`request#ShipmentLabels`:** This service is responsible for requesting shipping labels for a shipment. It retrieves the necessary gateway details and then calls the `requestLabelsServiceName` to interact with the specific shipping provider's API.

*   **`refund#ShipmentLabels` and `track#ShipmentLabels`:** These services follow a similar pattern to `request#ShipmentLabels`, retrieving gateway details and calling the appropriate service to handle refunds or tracking.

*   **`validate#PostalAddress`:** This service is a wrapper around the `validate#ShippingPostalAddress` interface. It handles the logic of determining which gateway configuration to use for validation and then calls the corresponding service.


## System setup guide

Here's sample data to setup FedEx gateway and a detailed system setup guide:

**1. Enumeration Records**

*   These records define the necessary configuration options for the FedEx gateway:
    *   `SgoFedExClientId`: Stores the FedEx REST API Client ID.
    *   `SgoFedExClientSecret`: Stores the FedEx REST API Client Secret.
    *   `SgoFedExAccountNumber`: Stores the FedEx Account Number.
    *   `ShGtwyFedEx`: Identifies the FedEx REST API as the shipping gateway type.
    *   `EgFedExOption`: Defines an enumeration group for FedEx gateway options.

**2. ShipmentBoxType Records**

*   These records define the various box types supported by FedEx, including their dimensions and corresponding gateway box IDs. This information is crucial for accurate rate calculation and label generation.

**3. ShippingGatewayConfig Record**

*   This record is the core configuration for the FedEx shipping gateway:
    *   `shippingGatewayConfigId`: "FedEx_DEMO" (a unique identifier for this configuration).
    *   `shippingGatewayTypeEnumId`: "ShGtwyFedEx" (indicates that this is a FedEx gateway).
    *   `description`: "FedEx API Demo" (a brief description).
    *   `getRateServiceName`: "mantle.FedExServices.get#ShippingRate" (the service used to get shipping rates).
    *   `requestLabelServiceName`: "mantle.FedExServices.create#ShippingLabel" (the service used to create shipping labels).
    *   **`methods` (child elements):** Map Moqui's standard shipment methods to FedEx's specific service codes.
    *   **`options` (child elements):** Store the FedEx API credentials and label type preferences.

**4. PartySetting Record**

*   This record sets the default shipping gateway for the organization "ORG_ZIZI_RETAIL" to the "FedEx_DEMO" configuration.

**System Setup Guide**


1.  **Data Import:**
    *   Prepare and Import the configuration XML data into your database. This will create the necessary records in the `moqui.basic.Enumeration`, `mantle.shipment.ShipmentBoxType`, `mantle.shipment.carrier.ShippingGatewayConfig`, and `mantle.party.PartySetting` entities.

2.  **Service Implementation:**
    *   Implement the Moqui services referenced in the `ShippingGatewayConfig` record:
        *   `mantle.FedExServices.get#ShippingRate`
        *   `mantle.FedExServices.create#ShippingLabel`
    *   These services should handle the communication with the FedEx API to retrieve rates and generate labels.

3.  **Configuration:**
    *   Update the `optionValue` fields in the `ShippingGatewayConfig.options` records with your actual FedEx API credentials.

**Important Note**
We will copy over limited set entity definition and services from Moqui UDM and USL. 

### Useful links

*   https://github.com/moqui/mantle-shippo/tree/master
*   https://github.com/hotwax/mantle-fedex
*   https://github.com/hotwax/mantle-shipstation
*   https://github.com/hotwax/mantle-shipengine


```xml
    <mantle.party.PartySetting partyId="ORG_ZIZI_RETAIL" partySettingTypeId="ValidateAddressGatewayConfigId" settingValue="SHIPPO_DEMO"/>
    <mantle.party.PartySetting partyId="ORG_ZIZI_RETAIL" partySettingTypeId="DefaultShipmentGatewayConfigId" settingValue="SHIPPO_DEMO"/>
```

This data defines party-specific settings for address validation and default shipment gateway. Let's break down each part:

*   `mantle.party.PartySetting`: This entity is used to store various settings and preferences for different parties in Moqui. In this case, it's configuring settings for a party with `partyId = "ORG_ZIZI_RETAIL"`. This represents a retail organization the demo data.

*   `partySettingTypeId`: This field indicates the type of setting being configured. There are two settings being defined here:

    *   `ValidateAddressGatewayConfigId`: This setting specifies which shipping gateway configuration should be used for address validation. The `settingValue = "SHIPPO_DEMO"` indicates that the "SHIPPO_DEMO" `ShippingGatewayConfig` (which we discussed earlier) should be used for validating addresses for this party.

    *   `DefaultShipmentGatewayConfigId`: This setting determines the default shipping gateway configuration to be used for this party. Again, `settingValue = "SHIPPO_DEMO"` means that Shippo will be the default gateway for shipments associated with this party.

*   Additional context:
    *   These settings will be used as defaults if there are no specific settings defined at the store level (referring to a retail store or online store).
    *   The screens where a store or vendor is not explicitly specified, the "Owner Party" (likely the party who owns or manages the Moqui instance) must be set to use these settings.



```
 <mantle.product.store.ProductStore productStoreId="POPC_DEFAULT">
        <shipOptions carrierPartyId="_NA_" shipmentMethodEnumId="ShMthGround" sequenceNum="1"/>
        <shipOptions carrierPartyId="USPS" shipmentMethodEnumId="ShMthGround" sequenceNum="5"/>
        <shipOptions carrierPartyId="USPS" shipmentMethodEnumId="ShMthThirdDay" sequenceNum="6"/>
        <shipOptions carrierPartyId="USPS" shipmentMethodEnumId="ShMthNextDay" sequenceNum="7"/>
        <shipOptions carrierPartyId="UPS" shipmentMethodEnumId="ShMthGround" sequenceNum="11"/>
        <shipOptions carrierPartyId="UPS" shipmentMethodEnumId="ShMthThirdDay" sequenceNum="12"/>
        <shipOptions carrierPartyId="UPS" shipmentMethodEnumId="ShMthSecondDay" sequenceNum="13"/>
        <shipOptions carrierPartyId="UPS" shipmentMethodEnumId="ShMthNextDay" sequenceNum="14"/>
        <shipOptions carrierPartyId="FedEx" shipmentMethodEnumId="ShMthGround" sequenceNum="21"/>
        <shipOptions carrierPartyId="FedEx" shipmentMethodEnumId="ShMthThirdDay" sequenceNum="22"/>
        <shipOptions carrierPartyId="FedEx" shipmentMethodEnumId="ShMthSecondDay" sequenceNum="23"/>
        <shipOptions carrierPartyId="FedEx" shipmentMethodEnumId="ShMthNextDay" sequenceNum="24"/>
        <shippingGateways carrierPartyId="_NA_" shippingGatewayConfigId="NA_LOCAL"/>
        <shippingGateways carrierPartyId="USPS" shippingGatewayConfigId="SHIPPO_DEMO"/>
        <shippingGateways carrierPartyId="UPS" shippingGatewayConfigId="SHIPPO_DEMO"/>
        <shippingGateways carrierPartyId="FedEx" shippingGatewayConfigId="SHIPPO_DEMO"/>
    </mantle.product.store.ProductStore>
```
This XML snippet demonstrates how to configure shipping options and gateway preferences for a specific product store in Moqui. It provides a template for customizing shipping behavior at the store level.

Here's a breakdown of the data:

*   `mantle.product.store.ProductStore`: This entity represents a product store, which could be a physical retail store or an online store. This configuration is for a store with `productStoreId = "POPC_DEFAULT"`, likely a default or primary store in the demo data.

*   `shipOptions`: This element is used to specify the preferred order of shipping methods for different carriers. Each `shipOptions` entry includes:
    *   `carrierPartyId`: The ID of the carrier (or "_NA_" for any carrier).
    *   `shipmentMethodEnumId`: The ID of the shipment method type from the `ShipmentMethod` enumeration.
    *   `sequenceNum`: A number to determine the order in which the shipping methods should be displayed or considered for this store. Lower numbers have higher priority.

    For example, `shipOptions carrierPartyId="USPS" shipmentMethodEnumId="ShMthGround" sequenceNum="5"` indicates that USPS Ground should have a sequence number of 5 for this store.

*   `shippingGateways`: This element associates specific carriers with shipping gateway configurations for this store. Each `shippingGateways` entry includes:
    *   `carrierPartyId`: The ID of the carrier (or "_NA_" for any carrier).
    *   `shippingGatewayConfigId`: The ID of the `ShippingGatewayConfig` to use for this carrier.

    For instance, `shippingGateways carrierPartyId="FedEx" shippingGatewayConfigId="SHIPPO_DEMO"` means that FedEx shipments for this store should be processed through the "SHIPPO_DEMO" gateway configuration.

