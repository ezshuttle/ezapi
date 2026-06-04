# EzShuttle QuickBookings API v2.1

## 📋 Table of Contents

- [1. Base API URL](#1-base-api-url)
- [2. Authorization](#2-authorization)
- [3. LocationFinder](#3-locationfinder)
- [4. QuickQuotes](#4-quickquotes)
- [5. QuickMultiQuotes](#5-quickmultiquotes)
- [6. QuickBookings (Create)](#6-quickbookings-create)
- [7. QuickBookings (Cancel)](#7-quickbookings-cancel)
- [8. QuickBookings (Get)](#8-quickbookings-get)
- [9. QuickConfirmations](#9-quickconfirmations)
- [10. UpdatePurchaseOrderRequest](#10-updatepurchaseorderrequest)
- [11. QuickClientReferenceLookup](#11-quickclientreferencelookup)
- [12. Error Codes](#12-error-codes)
- [13. Webhook](#13-webhook)

---

## 1. Base API URL

All endpoints described in this document (except LocationFinder) use the following base API URL:

```
https://api.ezshuttle.co.za/ezx/quickbooking/api/
```

---

## 2. Authorization

API authorization uses **HTTP Basic Authentication** for compatibility with legacy partner applications.

- **Username/Password**: Same credentials used for the [Client Portal](https://clientportal.ezshuttle.co.za)
- **New Account**: Click "Want To Open An Account" on the Client Portal page

---

## 3. LocationFinder

### Overview

Our endpoints accept either:
- **Google PlaceId's** (Google PID - Google Places AutoComplete)
- **EzShuttle PlaceId's** (EzPID - EzShuttle API Placefinder)

> **💡 Recommendation**: Use EzPID's with our place finder endpoint for:
> - Automatically filtered locations we service
> - Built-in IATA code handling for airports
> - Lower latency for quotes and reservations

### Implementation

Use a library like [TarekRaafat/autoComplete](https://github.com/TarekRaafat/autoComplete.js) for the LocationFinder.  
See basic working demo (jQuery) under `Samples/LocationFinder`

### AutoComplete Service

**Endpoint**: `GET /AutoComplete`  
**Base URL**: `https://api.ezshuttle.co.za/ezx/locationfinder/api/`  
**Headers**: `Content-Type: application/json`

#### QueryString Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `querytext` | string | ✓ | Query text |
| `sessionid` | string | ✓ | Unique session identifier |
| `partnerKey` | string | ✓ | Supplied on account creation |

#### Response
```json
{
    "displayName": "Sandton City, Rivonia Road, Sandhurst, Sandton, South Africa",
    "googlePlaceId": "ChIJu6K6ei1zlR4RS3iNn8RaO24"
}
```

### GooglePlaceIdLookup Service

**Endpoint**: `GET /GooglePlaceIdLookup`  
**Base URL**: `https://api.ezshuttle.co.za/ezx/locationfinder/api/`  
**Headers**: `Content-Type: application/json`

#### QueryString Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `googlePlaceId` | string | ✓ | Google PlaceId from AutoComplete OR IATA code |
| `sessionId` | string | ✓ | Same session ID used for AutoComplete |

#### IATA Format (Special Case for Airports)
```
|IATA|<code>
```
Example: `|IATA|PLZ`

#### Response
Returns EzShuttle PlaceId (EzPid):
```
EZKnYwMDEqMnwtMjYuMTY1NTA4Nzk5OTk5OTl8MjguMTQwNzkxMnwtMXw0OXw3OSBCb2VpbmcgUmQgRSwgQmVkZm9yZHZpZXcsIEdlcm1pc3RvbiwgMjAwNywgU291dGggQWZyaWNhfDEwMDR8MXwxMC4yNzExfA==
```

---

## 4. QuickQuotes

Get a quick price quote for a single vehicle type.

**Endpoint**: `GET /QuickQuotes/get`  
**Headers**: `Content-Type: application/json`

### QueryString Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pickupDateTime` | DateTime | ✓ | SAST DateTime or UTC DateTime<br/>Format: `2025-07-29T07:00:00` or `2025-07-29T05:00:00Z` |
| `pickupPlaceId` | string | ✓ | Google PID or EzShuttle EzPID |
| `destinationPlaceId` | string | ✓ | Google PID or EzShuttle EzPID |
| `vehicleType` | int | ✓ | Vehicle type code |
| `numberOfPassengers` | int | ✓ | Number of passengers (min=1, max=13) |
| `includeBabySeat` | bool | ✓ | Baby seat required |
| `includeTrailer` | bool | ✓ | Trailer required |
| `returnPickupDateTime` | DateTime | - | Return trip datetime (optional) |

### Response
**Headers**: `Content-Type: text/plain`  
**Body**: `500000` (Amount in ZAR cents - R500.00 in this example)

### Recommended Vehicle Type Codes
| Code | Capacity | Description |
|------|----------|-------------|
| 1 | 3 pax | Sedan |
| 3 | 9 pax | Minibus 9 pax |
| 4 | 13 pax | Minibus 13 pax |

> **📌 Note**: Vehicle types can be retrieved dynamically from:  
> `https://api.ezshuttle.co.za/ezx/quote/api/VehicleType` (includes thumbnails)

---

## 5. QuickMultiQuotes

Get quotes for multiple vehicle types in a single request.

**Endpoint**: `GET /QuickMultiQuotes/get`  
**Headers**: `Content-Type: application/json`

### QueryString Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pickupDateTime` | DateTime | ✓ | UTC DateTime (timezone info stripped)<br/>Format: `2025-07-29T07:00:00` |
| `pickupPlaceId` | string | ✓ | Google PID or EzShuttle EzPID |
| `destinationPlaceId` | string | ✓ | Google PID or EzShuttle EzPID |
| `numberOfPassengers` | int | ✓ | Number of passengers (min=1, max=13) |
| `includeBabySeat` | bool | ✓ | Baby seat required |
| `includeTrailer` | bool | ✓ | Trailer required |
| `returnPickupDateTime` | DateTime | - | Return trip datetime (optional) |
| `includeAvailabilityCheck` | bool | - | Conduct availability check (default: false) |

### Response
```json
[
    {
        "PrimaryTripQuoteId": "d941c276-062c-48df-9d2b-601db2574966",
        "ReturnTripQuoteId": "c8b58b75-5337-4f64-9c42-f9fa94b7274e",
        "TotalAmountInCents": 70000,
        "VehicleTypeId": 1,
        "VehicleTypeName": "Comfort - Sedan (3 seater)",
        "TermsAndConditionsUrl": "https://www.ezshuttle.co.za/faq/",
        "CancelAllowedDeadlineMinutes": 30,
        "CancelBillableDeadlineMinutes": 120,
        "IsAvailableForOnlinebookingPrimary": true,
        "IsAvailableForOnlinebookingReturn": true
    }
]
```

---

## 6. QuickBookings (Create)

Create a new booking reservation.

**Endpoint**: `POST /QuickBookings/post`  
**Headers**: `Content-Type: application/json`

### Request Body
```json
{
  "PickupPlaceId": "ChIJwymiBTgUlR4R1iEoeUAcv7M",
  "DestinationPlaceId": "ChIJ3XLuZMcPlR4RXSWvBLcK5o8",
  "PickupDisplayAddress": "",
  "DestinationDisplayAdadress": "entrance 4",
  "PickupFlightNumber": "FLTEST",
  "ReturnPickupFlightNumber": null,
  "PickupDateTime": "2025-07-28T07:00:00",
  "ReturnPickupDateTime": null,
  "NumberOfPassengers": 1,
  "VehicleType": 1,
  "IncludeBabySeat": false,
  "IncludeTrailer": false,
  "SpecialInstructions": null,
  "PaymentMethod": 4,
  "Name": "Richard",
  "Surname": "McIntyre",
  "Cell": "+27722939392",
  "Email": "richard@mailinator.com",
  "PurchaseOrder": "POTEST",
  "CostCentre": "COTEST",
  "ClientReservationId": "PR884526"
}
```

### Field Descriptions
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `PickupPlaceId` | string | ✓ | Google PID or EzShuttle EzPID |
| `DestinationPlaceId` | string | ✓ | Google PID or EzShuttle EzPID |
| `PickupDisplayAddress` | string | - | Additional address description |
| `DestinationDisplayAdadress` | string | - | Additional address description |
| `PickupFlightNumber` | string | Conditional | Required if pickup is an airport |
| `ReturnPickupFlightNumber` | string | Conditional | Required if return pickup is an airport |
| `PickupDateTime` | DateTime | ✓ | UTC DateTime (timezone stripped) |
| `ReturnPickupDateTime` | DateTime | - | Return trip datetime |
| `NumberOfPassengers` | int | ✓ | Passengers (min=1, max=13) |
| `VehicleType` | int | ✓ | Vehicle type code |
| `IncludeBabySeat` | bool | ✓ | Baby seat required |
| `IncludeTrailer` | bool | ✓ | Trailer required |
| `SpecialInstructions` | string | - | Special instructions |
| `PaymentMethod` | int | ✓ | Payment method (4 = Account) |
| `Name` | string | ✓ | Passenger first name |
| `Surname` | string | ✓ | Passenger surname |
| `Cell` | string | ✓ | Mobile in international format |
| `Email` | string | ✓ | Valid email address |
| `PurchaseOrder` | string | - | Client-defined purchase order |
| `CostCentre` | string | - | Cost centre |
| `ClientReservationId` | string | Recommended | Unique client reference |

### Response
```json
{
    "ReferenceId": 884526,
    "PickupPlaceId": "ChIJwymiBTgUlR4R1iEoeUAcv7M",
    "DestinationPlaceId": "ChIJ3XLuZMcPlR4RXSWvBLcK5o8",
    "PickupDisplayAddress": "",
    "DestinationDisplayAddress": "entrance 4",
    "PickupFlightNumber": "FLTEST",
    "ReturnPickupFlightNumber": "",
    "PickupDateTime": "2025-07-28T07:00:00",
    "ReturnPickupDateTime": null,
    "NumberOfPassengers": 1,
    "VehicleType": 1,
    "IncludeBabySeat": false,
    "IncludeTrailer": false,
    "SpecialInstructions": null,
    "PaymentMethod": 4,
    "Name": "Richard",
    "Surname": "McIntyre",
    "Cell": "+27722939392",
    "Email": "richard@mailinator.com",
    "PurchaseOrder": "POTEST",
    "CostCentre": "COTEST",
    "CostInCents": 50000,
    "QuoteId": null,
    "ClientReservationId": "PR884526"
}
```

---

## 7. QuickBookings (Cancel)

Cancel an existing booking.

**Endpoint**: `DELETE /QuickBookings/delete`  
**Headers**: `Content-Type: application/json`

### QueryString Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reference` | int | ✓ | EzShuttle reference ID |

### Response
```
Reservation Has Been Deleted For Id = 123456
```

---

## 8. QuickBookings (Get)

Retrieve booking details.

**Endpoint**: `GET /QuickBookings/get`  
**Headers**: `Content-Type: application/json`

### QueryString Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reference` | int | ✓ | EzShuttle reference ID |

### Response
```json
{
    "ReferenceId": 884526,
    "PickupPlaceId": "ChIJwymiBTgUlR4R1iEoeUAcv7M",
    "DestinationPlaceId": "ChIJ3XLuZMcPlR4RXSWvBLcK5o8",
    "PickupDisplayAddress": "",
    "DestinationDisplayAddress": "entrance 4",
    "PickupFlightNumber": "FLTEST",
    "ReturnPickupFlightNumber": "",
    "PickupDateTime": "2025-07-28T07:00:00",
    "ReturnPickupDateTime": null,
    "NumberOfPassengers": 1,
    "VehicleType": 1,
    "IncludeBabySeat": false,
    "IncludeTrailer": false,
    "SpecialInstructions": null,
    "PaymentMethod": 4,
    "Name": "Richard",
    "Surname": "McIntyre",
    "Cell": "+27722939392",
    "Email": "richard@mailinator.com",
    "PurchaseOrder": "POTEST",
    "CostCentre": "COTEST",
    "CostInCents": 50000,
    "QuoteId": null,
    "ClientReservationId": "PR884526"
}
```

---

## 9. QuickConfirmations

Get a PDF confirmation for a booking.

**Endpoint**: `GET /QuickConfirmations/get?reference={reference}`  
**Response Headers**: `Content-Type: application/pdf`

### Response
Returns a PDF file containing the booking confirmation.

---

## 10. UpdatePurchaseOrderRequest

Update the purchase order for an existing booking.

**Endpoint**: `POST /UpdatePurchaseOrderRequest`  
**Headers**: `Content-Type: application/json`

### Request Body
```json
{
    "ReferenceId": 884526,
    "PurchaseOrder": "UpdatedPO"
}
```

### Response
```
True
```

---

## 11. QuickClientReferenceLookup

Look up bookings by client reference ID. Please use this function when a timeout on create trip occurs, to determine if the trip was created.

**Endpoint**: `GET /QuickClientReferenceLookup/get/{clientReservationId}`  

### URL/QueryString Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `clientReservationId` | string | ✓ | Client-defined reference from booking |


### Response
```
Comma Delimited String containing all ReservationId's found matching ClientReferenceId
"123456"

OR

"123456,789777,876666"
```

> **⚠️ Important Notes**:
> - Returns HTTP 404 if not found
> - Use this endpoint to confirm booking creation after timeout
> - Ensure `ClientReservationId` is unique when creating a reservation

---

## 12. Error Codes

Errors return HTTP 400 with error details in the response body:

```json
{
    "message": "QuickBookings => POST => Result = Error Occurred",
    "stackTrace": "",
    "errorCode": 83
}
```

### Validation Errors (40-99, 300)
| Code | Error | Description |
|------|-------|-------------|
| 46 | SMSOrEmailNotificationFailure | - |
| 47 | BlockTripLuxuryVehicle | - |
| 48 | BlockTripOver3Passengers | - |
| 49 | BlockTripUnder4Passengers | - |
| 50 | BlockTrip | - |
| 51 | DestinationDestionationTrip | - |
| 52 | VehicleTypeUnknown | - |
| 53 | Trip24Hours | - |
| 54 | NoDropOff | - |
| 55 | NoPickup | - |
| 56 | NoPassenger | - |
| 57 | ParentClientInvalid | - |
| 58 | ParentClientBooking | - |
| 59 | InvalidClientPassenger | - |
| 60 | PassengerNotFound | - |
| 61 | ClientNotFound | - |
| 62 | ReservationIdNotNull | - |
| 63 | ReservationIsNull | - |
| 64 | ReservationClientNoLongerExist | - |
| 65 | ReservationNotFound | - |
| 66 | ReservationIdNullOrZero | - |
| 67 | NotificationTypeNotSpecified | - |
| 68 | QuoteConvertedAlreadyOrNotFound | - |
| 69 | TripTimeInvalid | - |
| 70 | PickupTimeIsNull | - |
| 71 | SuburbsOfficesInvalid | - |
| 72 | PricePlanNotFound | - |
| 73 | DestinationInvalid | - |
| 74 | SuburbInvalid | - |
| 75 | SuburbDestinationOfficesInvalid | - |
| 76 | NoAddressSpecified | - |
| 77 | UnauthorizedVehicleType | - |
| 78 | CancellationDeadlineExpired | - |
| 79 | AdhocTripsInternalError | - |
| 80 | CannotCancelPaidTrip | - |
| 81 | ReservationPaymentNotCreditCard | - |
| 82 | TripOutOfRangeOfOffice | - |
| 83 | TripOnlineDeadline | - |
| 84 | JhbPtaSuburbToSuburbNotSupported | - |
| 85 | SuburbOrDestinationNotFound | - |
| 86 | TrailerRequiresAdditionalPax | - |
| 92 | CancelTripLimitExceeded | - |
| 94 | QuoteNotAvailable | - |
| 98 | GeneralValidationError | - |
| 300 | MultiTripReservationNotSupported | - |

### General Errors (99-1000)
| Code | Error | Description |
|------|-------|-------------|
| 99 | GeneralError | - |
| 100 | Unauthorized | - |
| 101 | InternalServerError | - |
| 102 | ResourceNotFound | - |
| 1000 | AccountBlocked | - |

### Process Errors (200-300)
| Code | Error | Description |
|------|-------|-------------|
| 200 | ErrorSendingSMSOrEmailNotification | - |
| 300 | ErrorProcessingPayment | - |

---

## 13. Webhook

### Setup
Contact your account manager to register a webhook URL for real-time trip updates.

**Example URL**: `https://www.mytravelcorp.com/api/ez_webhook`

### Webhook Message Format
```json
{
  "PNR": "T0M7CRU50",
  "UpdateTypeId": 6,
  "PickupDateTime": "2026-06-04T19:30:00Z",
  "PickupDisplayAddress": "Cape Town International Airport",
  "DropOffDisplayAddress": "Taj Hotel & Suites",
  "PriceInCents": 9990000,
  "DriverName": "Tom Cruise",
  "DriverEta": "2026-06-04T19:28:00Z",
  "DriverPositionLatitude": -33.918861,
  "DriverPositionLongitude": 18.423300,
  "TripProgressPercentage": 0.50,
  "PreviousTripStatus": "OnRoute",
  "TripStatus": "OnRoute",
  "VehicleRegistration": "CA T0M-CR5",
  "VehicleDescription": "Bugatti Veyron (Black)",
  "IsCancelled": false,
  "MessageCreatedDateTimeUtc": "2026-06-04T19:15:30Z"
}
```

### Understanding the `UpdateTypeId` field

The `"UpdateTypeId"` integer property in the webhook payload maps to the following status change event types:

| `UpdateTypeId` Value | Enum Name | Description |
| :---: | :--- | :--- |
| **1** | **Create** | New booking created |
| **2** | **PickupTimeChanged** | Pickup date or time modified |
| **3** | **PickupDropOffLocationChanged** | Pickup or drop-off location modified |
| **4** | **DriverNameChanged** | Driver assigned (typically ≤3 hours before trip) |
| **5** | **DriverEtaChanged** | Driver ETA updated |
| **6** | **DriverLocationChanged** | Driver position updated (trip is in progress) |
| **9** | **Cancelled** | Trip cancelled (supports cancellation without assigned driver) |
| **10** | **Driver5MinuteArrivalAlarm** | Driver is less than 5 minutes away from the pickup point |
| **11** | **TripEndEtaChange** | Trip end ETA changed |
| **12** | **TripProgressChanged** | Progress percentage updated (General client group `*` only) |
| **13** | **TripStatusChanged** | Status updated (General client group `*` only) |

---

### Currently Supported Updates via Webhook

Our system publishes messages for **all of the following update events**:

1. **Create** (1) - Triggered when a new trip booking is successfully created.
2. **PickupTimeChanged** (2) - Triggered when a trip's pickup time is updated.
3. **PickupDropOffLocationChanged** (3) - Triggered when a trip's destination drop-off location changes.
4. **DriverNameChanged** (4) - Triggered when a driver is assigned or reassigned to a trip.
5. **DriverEtaChanged** (5) - Triggered when the driver's ETA updates during routing.
6. **DriverLocationChanged** (6) - Triggered when the driver's GPS location updates while on route or on trip.
7. **Cancelled** (9) - Triggered when the client cancels the trip (now supports unassigned trips as well).
8. **Driver5MinuteArrivalAlarm** (10) - Triggered when the driver is within 5 minutes of arrival.
9. **TripEndEtaChange** (11) - Triggered when the estimated trip completion time updates.
