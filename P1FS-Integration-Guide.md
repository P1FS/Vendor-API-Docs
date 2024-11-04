# Priority One Financial Services Integration Guide

## Overview

Priority One Financial Services, Inc. (Priority One or P1) is a company that specializes in providing financing solutions for recreational dealerships across the United States, focusing on marine, RV, and power sports sectors. They operate as an outsourcing partner, enabling dealerships to offer financing options to their customers by handling credit application submissions, loan processing, and lender negotiations on behalf of the dealership. Priority One aims to simplify and streamline the financing process, making it easier for customers to secure loans for their recreational purchases while assisting dealerships in managing finance-related aspects efficiently.

## Integration Overview

The Priority One API offers a secure platform for third-party companies to integrate with our system. This RESTful API enables seamless communication via JSON-formatted POST requests,
allowing external partners to integrate credit application workflows into their own systems with minimal friction.

By leveraging this API third-party companies can submit application data and check application statuses without manual intervention. Our platform handles data validation, secure data transfer,
and response formatting, ensuring compliance and data integrity across all transactions. Designed with scalability and ease of use in mind, the Priority One API supports real-time data exchange
and is secured by API keys.

This overview is the foundational guide for implementing Priority One’s API within your system, enabling credit application processing that integrates with your
existing technology stack.

## Document Scope

This document provides a comprehensive guide to integrating with the Priority One API, designed for third-party developers and system integrators who seek to submit and manage credit application data seamlessly.
The document outlines the technical requirements, configuration steps, and detailed API endpoints necessary for successful integration, focusing on transmitting data securely and reliably within a RESTful API
architecture.

Specifically, the document covers:

1. Authentication and Security: Instructions for setting up secure access using API keys.
2. Endpoint Descriptions: Detailed explanations of each API endpoint, including parameters, expected input/output formats, and error handling.
3. Request and Response Formatting: Data schema for JSON requests and responses, ensuring proper alignment with Priority One’s data model.
4. Data Validation and Error Codes: Definitions of validation rules and error codes to facilitate efficient troubleshooting and data integrity.
5. Best Practices and Compliance: Recommended practices for secure, efficient API usage, focusing on industry compliance standards.

This document aims to serve as a standalone resource, equipping developers and integrators with all necessary information to configure, test, and deploy the Priority One API within their systems for
an optimized, secure integration experience.

## Security

For security purposes, access to the RESTful API will be controlled using the following safeguards:

- A non-enumerable 128-bit Globally Unique Identifier (GUID).
- Requests to Priority One will contain the Authorization header containing a token provided by calling an Authentication endpoint.
- Parameters for the Authentication endpoint are the ClientId (A GUID provided by Priority One) and a ClientSecret (A string provided by Priority One). Upon success, a Bearer token will be returned.
- For security purposes, the ClientSecret will not be stored by Priority One. If lost, one will be regenerated and securely sent to the developer.
- Authentication tokens have a 20 minute expiration, and are signed by Priority One.
- All calls are rate limited and their limits are defined later in this document.

## Base URL:

https://ds.p1fs.com

## Endpoint: Authentication

POST /Vendor/Auth

This endpoint is used to authenticate a vendor and retrieve a JWT access token.

| Parameter      | Type   | Required | Description                 |
| :--- | :--- | :--- | :--- |
| `ClientId`     | Guid   | Yes      | The vendor's client ID.     |
| `ClientSecret` | string | Yes      | The vendor's client secret. |

### Rate Limiting
10 requests per minute, or 60 per hour

### Example Request Body

```json
{
  "ClientId": "19D69D1A-97F8-4D9E-9B7A-B6DDD70B39C2",
  "ClientSecret": "your-client-secret"
}
```

### Example Response
- 200 OK: Success, with JSON object
- 400 Bad Request: Invalid request
- 401 Unauthorized: Authentication failure

```json
{
  "access_token": "eyJhb..."
}
```

## Endpoint: Submit Credit Application
POST /CreditApp
AUTHORIZATION Header retrieved from /Vendor/Auth

### Process

1. Acme needs to collect a Credit Application from the customer.
2. The DMS generates a GUID to uniquely identify the deal. It is a standard 128-bit integer represented with hexadecimal digits and dashes, no curly brace (ex: 1231a4fe-039a-4de5-b0ab-f50c7841c1ab). From here on out we will refer to this as the **DealID**.
3. The DMS constructs the JSON object matching the schema provided by Priority One, and POST it to /CreditApp

This endpoint allows third-party companies to submit credit application data. All data must follow JSON formatting noted below

| Parameter      | Type   | Required | Description                 |
| :--- | :--- | :--- | :--- |
| `DealID`     | Guid   | Yes      | A GUID generated by the Vendor, which can be used later as a way to send/retrieve data on this deal.     |

### Rate Limiting
10 requests per minute, or 60 per hour

### Deal Object Schema

| Property   | Type           | Required | Description                                   |
|------------|----------------|----------|-----------------------------------------------|
| `Buyer`    | Applicant         | No       | Applicant details for the buyer.              |
| `CoBuyer`  | Applicant         | No       | Applicant details for the co-buyer.           |
| `DealInfo` | DealInfo         | No       | Information related to the deal.              |

### Applicant Object

| Property               | Type     | Required | Description                                   |
|------------------------|----------|----------|-----------------------------------------------|
| `RelationshipToBuyer`  | string   | No       | Relationship to the buyer. Allowed values: `Spouse`, `Significant Other`, `Parent`, `Sibling`, `Grand Parent`, `Family Member`, `Fiance`, `Boyfriend`, `Girlfriend`, `Other`. |
| `FirstName`            | string   | Yes      | Applicant's first name. Max length: 20.       |
| `MiddleName`           | string   | No       | Applicant's middle name. Max length: 20.      |
| `LastName`             | string   | Yes      | Applicant's last name. Max length: 20.        |
| `Suffix`               | string   | No       | Applicant's Suffix. Allowed values: `Jr`, `Sr`, `I`, `II`, `III`, `IV`. |
| `Zip`                  | string   | Yes      | Zip code. Must be exactly 5 characters long.  |
| `City`                 | string   | Yes      | City name. Max length: 25.                    |
| `State`                | string   | Yes      | State name. Min length 2. Max length: 50.                   |
| `StreetAddress`        | string   | Yes      | Street address. Max length: 50.               |
| `YearsAtResidence`     | int      | Yes      | Years at current residence. Range: 0-99.      |
| `MonthsAtResidence`    | int      | Yes      | Months at current residence. Range: 0-11.     |
| `PreviousZip`          | string   | No       | Previous zip code. Must be exactly 5 characters long. |
| `PreviousCity`         | string   | No       | Previous city name. Max length: 25.           |
| `PreviousState`        | string   | No       | Previous state name. Max length: 50.          |
| `PreviousStreetAddress`| string   | No       | Previous street address. Max length: 50.      |
| `PreviousYearsAtResidence`| int   | No       | Previous years at residence. Range: 0-99.     |
| `PreviousMonthsAtResidence`| int  | No       | Previous months at residence. Range: 0-11.    |
| `USCitizen`            | bool     | Yes      | Indicates if the applicant is a US citizen.   |
| `PhoneNumber`          | string   | Yes      | Phone number. Must be exactly 10 digits.      |
| `PhoneType`            | string   | Yes      | Type of phone. Allowed values: `Mobile`, `Landline`. |
| `Email`                | string   | Yes      | Email address. Max length: 50.                |
| `PreferredAlertMethod` | string   | Yes      | Preferred alert method. Allowed values: `Mobile`, `Email`. |
| `SSN`                  | string   | Yes      | Social Security Number. Must be exactly 9 digits. |
| `DriversLicense`       | string   | Yes      | Driver's license number. Max length: 20.      |
| `DOB`                  | datetime | Yes      | Date of birth. ISO-8601 Ex. `1950-05-25`                               |
| `HousingStatus`        | string   | Yes      | Housing status. Allowed values: `Buying`, `Free & clear`, `Renting`, `Living with parents`, `Other`. |
| `MonthlyHousingPayment`| decimal  | Yes      | Monthly housing payment. Range: 0 - 9,999,999.99. |
| `HousingDescription`   | string   | No       | Description of housing. Max length: 30.       |
| `EmploymentStatus`     | string   | Yes      | Employment status. Allowed values: `Unemployed`, `Employed`, `Self-Employed`, `Retired`. |
| `LengthOfRetirement`   | int      | No       | Length of retirement in years. Range: 0-99.   |
| `EmployerName`         | string   | No       | Name of the employer. Max length: 30.         |
| `EmployerPhoneNumber`  | string   | No       | Employer's phone number. Must be exactly 10 digits. |
| `JobTitle`             | string   | No       | Job title. Max length: 30.                    |
| `GrossMonthlyIncome`   | decimal  | No       | Gross monthly income. Range: 0 - 9,999,999.99.|
| `LengthOfEmploymentYears` | int   | No       | Length of employment in years. Range: 0-99.   |
| `LengthOfEmploymentMonths` | int  | No       | Length of employment in months. Range: 0-11.  |
| `PreviousEmployerName` | string   | No       | Previous employer's name. Max length: 30.     |
| `PreviousJobTitle`     | string   | No       | Previous job title. Max length: 30.           |
| `PreviousLengthOfEmploymentYears`| int | No  | Previous length of employment in years. Range: 0-99. |
| `PreviousLengthOfEmploymentMonths`| int | No | Previous length of employment in months. Range: 0-11. |
| `OtherIncome`          | IncomeSource[]    | No       | List of other income sources.                 |

### IncomeSource Object

| Property          | Type     | Required | Description                                   |
|-------------------|----------|----------|-----------------------------------------------|
| `Category`        | string   | Yes      | Allowed values: `Pension`, `Social Security`, `Disability`, `Investments`, `Alimony`, `Child Support`, `Other`. |
| `GrossMonthlyIncome` | decimal| Yes     | Gross monthly income from other sources. Range: 0 - 9,999,999.99. |

### DealInfo Object

| Property          | Type       | Required | Description                                   |
|-------------------|------------|----------|-----------------------------------------------|
| `PartnerID`       | Guid       | Yes      | Priority One assigned Guid for Dealer         |
| `DealerName`      | string     | No       | Name of the dealer. Max length: 40.           |
| `SalesPersonName` | string     | No       | Name of the salesperson. Max length: 25.      |
| `CollateralType`  | string     | Yes      | Collateral type. Allowed values: `5th Wheel`, `Motor Home`, `Tear Drop`, `Popup Camper`, `Travel Trailer`, `Park Trailer`, `Slide In`, `Cargo Utility`, `ATV`, `UTV`, `Horse Trailer With Living Quarters`, `Horse Trailer Without Living Quarters`, `Boat`, `Park Model`, `PWC`. |
| `ConditionOfPurchase` | string | Yes      | Condition of purchase. Allowed values: `New`, `Used`. |
| `Make`            | string     | Yes      | Make of the collateral. Max length: 30.       |
| `Model`           | string     | Yes      | Model of the collateral. Max length: 30.      |
| `Year`            | int        | Yes      | Year of the collateral. Range: 1500-3999.     |
| `VIN`             | string     | No       | Vehicle Identification Number. Max length: 50.|
| `DownPayment`     | decimal    | No       | Down payment amount. Range: 0 - 9,999,999.99. |
| `DateOfDelivery`  | datetime   | No       | Date of delivery. ISO-8601 Ex. `1950-05-25`   |
| `PreviouslyOwned` | bool       | No       | Indicates if previously owned.                |
| `TradeMake`       | string     | No       | Make of the trade. Max length: 30.            |
| `TradeModel`      | string     | No       | Model of the trade. Max length: 30.           |
| `TradeYear`       | int        | No       | Year of the trade. Range: 1500-3999.          |
| `TradeAllowanceAmount` | decimal | No     | Trade allowance amount. Range: 0 - 9,999,999.99. |
| `EstimateTradeDebt` | decimal  | No       | Estimated trade debt. Range: 0 - 9,999,999.99.|
| `PriceOfUnit`     | decimal    | No       | Price of the unit. Range: 0 - 9,999,999.99.   |
| `SalesTaxRate`    | decimal    | No       | Sales tax rate. Range: 0-20.                  |
| `SalesTaxValue`   | decimal    | No       | Sales tax value. Range: 0 - 9,999,999.99.     |
| `MiscFees`        | decimal    | No       | Miscellaneous fees. Range: 0 - 9,999,999.99.  |
| `AskAQuestion`    | string     | No       | Additional questions.                         |

### Example Request Body
```json
{
  "Buyer": {
    "FirstName": "John",
    "LastName": "Doe",
    "Zip": 12345,
    "City": "Schenectady",
    "State": "New York",
    "StreetAddress": "123 Test St",
    "PhoneNumber": "5551234567",
    "PhoneType": "Mobile",
    "Email": "example@p1fs.com",
    "PreferredAlertMethod": "Mobile",
    "SSN": "999123456",
    "DriversLicense": "DR1V3R",
    "DOB": "1996-04-27",
    "USCitizen": true,
    "HousingStatus": "Buying",
    "MonthlyHousingPayment": 2200.49,
    "EmploymentStatus": "Employed",
    "EmployerName": "Acme Boats",
    "EmployerPhoneNumber": "5557654321",
    "JobTitle": "Tester",
    "GrossMonthlyIncome": 6500.50,
    "LengthOfEmploymentYears": 12,
    "LengthOfEmploymentMonths": 3
  },
  "DealInfo": {
    "PartnerID": "1DA192E7-3AF7-437E-B7A5-C508D83B1000",
    "CollateralType": "Tear Drop",
    "ConditionOfPurchase": "New",
    "Make": "TestMake",
    "Model": "TestModel",
    "Year": 2024
  }
}
```

### Example Response
- 200 OK: Success, with JSON object
- 400 Bad Request: Invalid request
- 401 Unauthorized: Authentication failure
- 404 Not Found: Vendor or Dealer not found

```json
{
  "Success": true,
  "DealID": "8A7EF792-36C5-4BEF-B1C2-BC6B39496593"
}
```

## Endpoint: Add CoBuyer to Existing Credit Application
POST /CreditApp/CoBuyer 
AUTHORIZATION Header retrieved from /Vendor/Auth

### Process

1. Acme has already posted Credit Application to Priority One and received successful response.
2. Acme retrieves CoBuyer applicant information and constructs JSON object matching Priority One's schema.
3. Acme posts object to /CreditApp/CoBuyer and the DealID, which was previously constructed for the initial CreditApp POST.

This endpoint allows third-party companies to append a CoBuyer onto an already existing credit application.

| Parameter      | Type   | Required | Description                 |
| :--- | :--- | :--- | :--- |
| `DealID`     | Guid   | Yes      | A GUID generated by the Vendor, which can be used later as a way to send/retrieve data on this deal.     |

### Rate Limiting
10 requests per minute, or 60 per hour

### CoBuyer Schema

| Property   | Type           | Required | Description                                   |
|------------|----------------|----------|-----------------------------------------------|
| `CoBuyer`  | Applicant         | No       | Applicant details for the co-buyer.        |

### Applicant Object

| Property               | Type     | Required | Description                                   |
|------------------------|----------|----------|-----------------------------------------------|
| `RelationshipToBuyer`  | string   | Yes      | Relationship to the buyer. Allowed values: `Spouse`, `Significant Other`, `Parent`, `Sibling`, `Grand Parent`, `Family Member`, `Fiance`, `Boyfriend`, `Girlfriend`, `Other`. |
| `FirstName`            | string   | Yes      | Applicant's first name. Max length: 20.       |
| `MiddleName`           | string   | No       | Applicant's middle name. Max length: 20.      |
| `LastName`             | string   | Yes      | Applicant's last name. Max length: 20.        |
| `Suffix`               | string   | No       | Applicant's Suffix. Allowed values: `Jr`, `Sr`, `I`, `II`, `III`, `IV`. |
| `Zip`                  | string   | Yes      | Zip code. Must be exactly 5 characters long.  |
| `City`                 | string   | Yes      | City name. Max length: 25.                    |
| `State`                | string   | Yes      | State name. Min length 2. Max length: 50.                   |
| `StreetAddress`        | string   | Yes      | Street address. Max length: 50.               |
| `YearsAtResidence`     | int      | Yes      | Years at current residence. Range: 0-99.      |
| `MonthsAtResidence`    | int      | Yes      | Months at current residence. Range: 0-11.     |
| `PreviousZip`          | string   | No       | Previous zip code. Must be exactly 5 characters long. |
| `PreviousCity`         | string   | No       | Previous city name. Max length: 25.           |
| `PreviousState`        | string   | No       | Previous state name. Max length: 50.          |
| `PreviousStreetAddress`| string   | No       | Previous street address. Max length: 50.      |
| `PreviousYearsAtResidence`| int   | No       | Previous years at residence. Range: 0-99.     |
| `PreviousMonthsAtResidence`| int  | No       | Previous months at residence. Range: 0-11.    |
| `USCitizen`            | bool     | Yes      | Indicates if the applicant is a US citizen.   |
| `PhoneNumber`          | string   | Yes      | Phone number. Must be exactly 10 digits.      |
| `PhoneType`            | string   | Yes      | Type of phone. Allowed values: `Mobile`, `Landline`. |
| `Email`                | string   | Yes      | Email address. Max length: 50.                |
| `PreferredAlertMethod` | string   | Yes      | Preferred alert method. Allowed values: `Mobile`, `Email`. |
| `SSN`                  | string   | Yes      | Social Security Number. Must be exactly 9 digits. |
| `DriversLicense`       | string   | Yes      | Driver's license number. Max length: 20.      |
| `DOB`                  | datetime | Yes      | Date of birth. ISO-8601 Ex. `1950-05-25`                               |
| `HousingStatus`        | string   | Yes      | Housing status. Allowed values: `Buying`, `Free & clear`, `Renting`, `Living with parents`, `Other`. |
| `MonthlyHousingPayment`| decimal  | Yes      | Monthly housing payment. Range: 0 - 9,999,999.99. |
| `HousingDescription`   | string   | No       | Description of housing. Max length: 30.       |
| `EmploymentStatus`     | string   | Yes      | Employment status. Allowed values: `Unemployed`, `Employed`, `Self-Employed`, `Retired`. |
| `LengthOfRetirement`   | int      | No       | Length of retirement in years. Range: 0-99.   |
| `EmployerName`         | string   | No       | Name of the employer. Max length: 30.         |
| `EmployerPhoneNumber`  | string   | No       | Employer's phone number. Must be exactly 10 digits. |
| `JobTitle`             | string   | No       | Job title. Max length: 30.                    |
| `GrossMonthlyIncome`   | decimal  | No       | Gross monthly income. Range: 0 - 9,999,999.99.|
| `LengthOfEmploymentYears` | int   | No       | Length of employment in years. Range: 0-99.   |
| `LengthOfEmploymentMonths` | int  | No       | Length of employment in months. Range: 0-11.  |
| `PreviousEmployerName` | string   | No       | Previous employer's name. Max length: 30.     |
| `PreviousJobTitle`     | string   | No       | Previous job title. Max length: 30.           |
| `PreviousLengthOfEmploymentYears`| int | No  | Previous length of employment in years. Range: 0-99. |
| `PreviousLengthOfEmploymentMonths`| int | No | Previous length of employment in months. Range: 0-11. |
| `OtherIncome`          | IncomeSource[]    | No       | List of other income sources.                 |

### IncomeSource Object

| Property          | Type     | Required | Description                                   |
|-------------------|----------|----------|-----------------------------------------------|
| `Category`        | string   | Yes      | Allowed values: `Pension`, `Social Security`, `Disability`, `Investments`, `Alimony`, `Child Support`, `Other`. |
| `GrossMonthlyIncome` | decimal| Yes     | Gross monthly income from other sources. Range: 0 - 9,999,999.99. |

### Example Request Body
```json
{
  "CoBuyer": {
   "RelationshipToBuyer": "Spouse",
    "FirstName": "Jane",
    "LastName": "Doe",
    "Zip": 12345,
    "City": "Schenectady",
    "State": "New York",
    "StreetAddress": "123 Test St",
    "PhoneNumber": "5551234567",
    "PhoneType": "Mobile",
    "Email": "example@p1fs.com",
    "PreferredAlertMethod": "Mobile",
    "SSN": "999123456",
    "DriversLicense": "DR1V3R",
    "DOB": "1996-04-27",
    "USCitizen": true,
    "HousingStatus": "Buying",
    "MonthlyHousingPayment": 2200.49,
    "EmploymentStatus": "Employed",
    "EmployerName": "Acme Boats",
    "EmployerPhoneNumber": "5557654321",
    "JobTitle": "Tester",
    "GrossMonthlyIncome": 6500.50,
    "LengthOfEmploymentYears": 12,
    "LengthOfEmploymentMonths": 3
  }
}
```

### Example Response
- 200 OK: Success, with JSON object
- 400 Bad Request: Invalid request
- 401 Unauthorized: Authentication failure
- 404 Not Found: Vendor or Dealer not found

```json
{
  "Success": true,
  "DealID": "822D19BB-85E7-4A82-BF07-7DCB159EA93C"
}
```

## Endpoint: Get Deal
GET /Deal/
AUTHORIZATION Header retrieved from /Vendor/Auth

### Process

1. Acme has already posted Credit Application to Priority One and received successful response.
2. Acme wants to manually check the status of a deal.
3. Acme calls /Deal endpoint and receives deal.

This endpoint allows third-party companies to get the Deal info from Priority One using the DealID.

| Parameter      | Type   | Required | Description                 |
| :--- | :--- | :--- | :--- |
| `DealID`     | Guid   | Yes      | A GUID generated by the Vendor, which can be used later as a way to send/retrieve data on this deal.     |

### Rate Limiting
10 requests per minute, or 60 per hour

### CoBuyer Schema

| Property   | Type           | Required | Description                                   |
|------------|----------------|----------|-----------------------------------------------|
| `CoBuyer`  | Applicant         | No       | Applicant details for the co-buyer.        |

### Applicant Object

| Property               | Type     | Required | Description                                   |
|------------------------|----------|----------|-----------------------------------------------|
| `RelationshipToBuyer`  | string   | Yes      | Relationship to the buyer. Allowed values: `Spouse`, `Significant Other`, `Parent`, `Sibling`, `Grand Parent`, `Family Member`, `Fiance`, `Boyfriend`, `Girlfriend`, `Other`. |
| `FirstName`            | string   | Yes      | Applicant's first name. Max length: 20.       |
| `MiddleName`           | string   | No       | Applicant's middle name. Max length: 20.      |
| `LastName`             | string   | Yes      | Applicant's last name. Max length: 20.        |
| `Suffix`               | string   | No       | Applicant's Suffix. Allowed values: `Jr`, `Sr`, `I`, `II`, `III`, `IV`. |
| `Zip`                  | string   | Yes      | Zip code. Must be exactly 5 characters long.  |
| `City`                 | string   | Yes      | City name. Max length: 25.                    |
| `State`                | string   | Yes      | State name. Min length 2. Max length: 50.                   |
| `StreetAddress`        | string   | Yes      | Street address. Max length: 50.               |
| `YearsAtResidence`     | int      | Yes      | Years at current residence. Range: 0-99.      |
| `MonthsAtResidence`    | int      | Yes      | Months at current residence. Range: 0-11.     |
| `PreviousZip`          | string   | No       | Previous zip code. Must be exactly 5 characters long. |
| `PreviousCity`         | string   | No       | Previous city name. Max length: 25.           |
| `PreviousState`        | string   | No       | Previous state name. Max length: 50.          |
| `PreviousStreetAddress`| string   | No       | Previous street address. Max length: 50.      |
| `PreviousYearsAtResidence`| int   | No       | Previous years at residence. Range: 0-99.     |
| `PreviousMonthsAtResidence`| int  | No       | Previous months at residence. Range: 0-11.    |
| `USCitizen`            | bool     | Yes      | Indicates if the applicant is a US citizen.   |
| `PhoneNumber`          | string   | Yes      | Phone number. Must be exactly 10 digits.      |
| `PhoneType`            | string   | Yes      | Type of phone. Allowed values: `Mobile`, `Landline`. |
| `Email`                | string   | Yes      | Email address. Max length: 50.                |
| `PreferredAlertMethod` | string   | Yes      | Preferred alert method. Allowed values: `Mobile`, `Email`. |
| `SSN`                  | string   | Yes      | Social Security Number. Must be exactly 9 digits. |
| `DriversLicense`       | string   | Yes      | Driver's license number. Max length: 20.      |
| `DOB`                  | datetime | Yes      | Date of birth. ISO-8601 Ex. `1950-05-25`                               |
| `HousingStatus`        | string   | Yes      | Housing status. Allowed values: `Buying`, `Free & clear`, `Renting`, `Living with parents`, `Other`. |
| `MonthlyHousingPayment`| decimal  | Yes      | Monthly housing payment. Range: 0 - 9,999,999.99. |
| `HousingDescription`   | string   | No       | Description of housing. Max length: 30.       |
| `EmploymentStatus`     | string   | Yes      | Employment status. Allowed values: `Unemployed`, `Employed`, `Self-Employed`, `Retired`. |
| `LengthOfRetirement`   | int      | No       | Length of retirement in years. Range: 0-99.   |
| `EmployerName`         | string   | No       | Name of the employer. Max length: 30.         |
| `EmployerPhoneNumber`  | string   | No       | Employer's phone number. Must be exactly 10 digits. |
| `JobTitle`             | string   | No       | Job title. Max length: 30.                    |
| `GrossMonthlyIncome`   | decimal  | No       | Gross monthly income. Range: 0 - 9,999,999.99.|
| `LengthOfEmploymentYears` | int   | No       | Length of employment in years. Range: 0-99.   |
| `LengthOfEmploymentMonths` | int  | No       | Length of employment in months. Range: 0-11.  |
| `PreviousEmployerName` | string   | No       | Previous employer's name. Max length: 30.     |
| `PreviousJobTitle`     | string   | No       | Previous job title. Max length: 30.           |
| `PreviousLengthOfEmploymentYears`| int | No  | Previous length of employment in years. Range: 0-99. |
| `PreviousLengthOfEmploymentMonths`| int | No | Previous length of employment in months. Range: 0-11. |
| `OtherIncome`          | IncomeSource[]    | No       | List of other income sources.                 |

### IncomeSource Object

| Property          | Type     | Required | Description                                   |
|-------------------|----------|----------|-----------------------------------------------|
| `Category`        | string   | Yes      | Allowed values: `Pension`, `Social Security`, `Disability`, `Investments`, `Alimony`, `Child Support`, `Other`. |
| `GrossMonthlyIncome` | decimal| Yes     | Gross monthly income from other sources. Range: 0 - 9,999,999.99. |

### Example Response
- 200 OK: Success, with JSON object
- 400 Bad Request: Invalid request
- 401 Unauthorized: Authentication failure
- 404 Not Found: Vendor or Dealer not found

```json
{
  "Success": true,
  "DealID": "40A5D3F5-7816-449D-A67B-052483FA9ADE"
}
```

## Deal Webhook

It is possible to setup a webhook for Priority One to POST Deals to whenever the deal updates on Priority One's side.

### Process
1. Provide Priority One with your webhook URL and a bearer token.
2. Priority One will POST the deal object to this webhook after the deal was updated, passing the bearer token in the Authorization header.

### Deal updates
The following interface defines the format of what P1 will send to the DMS when any of the following items on a deal changes:

- DealID
- Status
- Stage
- Detail
- Lender
- Staff
- Buyers Name
- Who's working the deal
- Phone number for who is working the deal

### Deal Format
| Property    | Type       | Required | Description                                     |
|-------------|------------|----------|-------------------------------------------------|
| `DealID`    | Guid       | Yes      | Unique identifier for the deal.                |
| `PartnerID` | Guid       | Yes      | Unique identifier for the partner.             |
| `DRCStatus` | string     | Yes      | Current status of the DRC.                     |
| `DRCStage`  | string     | Yes      | Current stage of the DRC.                      |
| `DRCDetail` | string     | Yes      | Additional detail about the DRC status.        |
| `Staff`     | Staff     | Yes      | Details of the staff member handling the deal. |
| `Buyer`     | Applicant     | Yes      | Details of the buyer.                          |
| `CoBuyer`   | Applicant     | No       | Details of the co-buyer, if applicable.        |
| `Collateral`| Collateral     | Yes      | Information about the collateral.              |

### Applicant Object

| Property    | Type   | Required | Description                                     |
|-------------|--------|----------|-------------------------------------------------|
| `FirstName` | string | Yes      | First name of the applicant.                   |
| `MiddleName`| string | No       | Middle name of the applicant.                  |
| `LastName`  | string | Yes      | Last name of the applicant.                    |
| `Generation`| string | No       | Generation suffix (e.g., Jr, Sr, III, etc.).   |
| `Address`   | string | Yes      | Street address of the applicant.               |
| `City`      | string | Yes      | City of the applicant.                         |
| `State`     | string | Yes      | State of the applicant.                        |
| `Zip`       | string | Yes      | Zip code of the applicant.                     |
| `HomePhone` | string | No       | Home phone number of the applicant.            |
| `CellPhone` | string | No       | Cell phone number of the applicant.            |

### Staff Object

| Property | Type   | Required | Description                                     |
|----------|--------|----------|-------------------------------------------------|
| `Name`   | string | Yes      | Name of the staff member.                      |
| `Phone`  | string | Yes      | Phone number of the staff member.              |

### Collateral Object

| Property         | Type   | Required | Description                                     |
|------------------|--------|----------|-------------------------------------------------|
| `Type`           | string | Yes      | Main type of the collateral (e.g., `RV`, `Boat`, `PWC`). |
| `CollateralSubType` | string | No   | Specific subtype of the collateral when applicable (e.g., `5th Wheel`, `Motor Home`, `Tear Drop`). |
| `Make`           | string | Yes      | Make of the collateral.                        |
| `Model`          | string | Yes      | Model of the collateral.                       |
| `Year`           | int | Yes      | Year of manufacture of the collateral.         |
| `IsNew`          | bool | Yes      | Indicates whether the collateral is new or used. |
| `SerialNumber`   | string | No      | Serial number of the collateral.               |
| `Motors`         | Motor[]  | No       | Array of Motor objects, if applicable.         |
| `Trailers`       | Trailer[]  | No       | Array of Trailer objects, if applicable.       |

### Motor Object

| Property       | Type   | Required | Description                                     |
|----------------|--------|----------|-------------------------------------------------|
| `Make`         | string | Yes      | Make of the motor.                             |
| `Model`        | string | Yes      | Model of the motor.                            |
| `Year`         | int | Yes      | Year of manufacture of the motor.              |
| `IsNew`        | bool | Yes      | Indicates whether the motor is new or used.    |
| `SerialNumber` | string | No      | Serial number of the motor.                    |
| `Power`        | string | No      | Power specification of the motor (e.g., 150HP).|
| `Type`         | string | No      | Type of the motor (e.g., Outboard, Inboard).   |

### Trailer Object

| Property       | Type   | Required | Description                                     |
|----------------|--------|----------|-------------------------------------------------|
| `Make`         | string | Yes      | Make of the trailer.                           |
| `Model`        | string | Yes      | Model of the trailer.                          |
| `Year`         | int | Yes      | Year of manufacture of the trailer.            |
| `IsNew`        | bool | Yes      | Indicates whether the trailer is new or used.  |
| `SerialNumber` | string | No      | Serial number of the trailer.                  |


### Example Post
```json
{
   "DealID": "19D69D1A-97F8-4D9E-9B7A-B6DDD70B39C2",
   "PartnerID": "7DE2EBF3-3DE9-4508-A338-9107A893D4E9",
   "DRCStatus": "Working",
   "DRCStage": "Working with Buyer",
   "DRCDetail": "Need to Interview for More Information",
   "Staff": {
      "Name": "Priority Doe",
      "Phone": "5557981234",
   },
   "Buyer": {
      "FirstName": "John",
      "LastName": "Doe",
      "MiddleName": null,
      "Generation": null,
      "Address": "123 Main Street",
      "City": "Saint Petersburg",
      "State": "NH",
      "Zip": "33710",
      "HomePhone": "5551234567",
      "CellPhone": "5557984561"
   },
   "CoBuyer": {
      "FirstName": "Jane",
      "LastName": "Doe",
      "MiddleName": null,
      "Generation": null,
      "Address": "123 Main Street",
      "City": "Saint Petersburg",
      "State": "NH",
      "Zip": "33710",
      "HomePhone": "5551234567",
      "CellPhone": null
   },
   "Collateral": {
      "Type": "Marine",
      "CollateralSubType": null,
      "Make": "Yamaha",
      "Model": "The Twelve Series",
      "Year": 2022,
      "IsNew": true,
      "SerialNumber": "329488MNMD88ABABA",
      "Motors": [
         {
            "Make": "MERCURY",
            "Model": "SUPERDUPER",
            "Year": 2019,
            "IsNew": false,
            "SerialNumber": "OG349288",
            "Power": "115",
            "Type": "Outboard 2 Stroke Carburetor"
         },
         {
            "Make": "MERCURY",
            "Model": "SUPERDUPER",
            "Year": 2022,
            "IsNew": true,
            "SerialNumber": "OG249211",
            "Power": "115",
            "Type": "Outboard 2 Stroke Carburetor"
         }
      ],
      "Trailers": [
         {
            "Make": "FLOAT ON",
            "Model": "927TXCBB",
            "Year": 2019,
            "IsNew": false,
            "SerialNumber": "XHO87473"
         }
      ]
   }
}
```