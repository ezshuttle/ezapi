# EzShuttle QuickBookings API v2.0

Sections

1. Base API Url
2. Authorization
3. LocationFinder
4. QuickQuotes
5. QuickBookings (Create)
6. QuickBookings (Cancel)
7. UpdatePurchaseOrderRequest
8. Error Codes

# 1 Base API Url

All endpoints described in this document with the exception of LocationFinder have the following base API url:
https://api.ezshuttle.co.za/ezx/quickbooking/api/

# 2 Authorization

API authorization is implemented with HTTP Basic Authentication in order to retain compatibility with certain legacy partner applications.
The username/password is the same as you would use to access the Client Portal: https://ezshuttle.co.za/bookings/ClientPortal/admin.shtml
Should you not have an account please apply for one by clicking "Want To Open An Account" on the above page.

# 3 LocationFinder

Our endpoints accept either Google PlaceId's (Google PID - Google Places AutoComplete) or EzShuttle PlaceId's (EzPID - EzShuttle API Placefinder). 
We recemend the use of EzPID's with our place finder endpoint for the following reasons:

a. Automaticlly filtered to only provide locations which we service.
b. Built in Handling of IATA codes for airport pickup's / dropoffs.
c. Lower latency for Quote Generation / Reservation Creation.

It is reccomended that a library such as https://github.com/TarekRaafat/autoComplete.js is used for the LocationFinder.
Please see basic working demo (JQUERY) under Samples/LocationFinder

AutoComplete Service

* Http Verb: GET
* Headers: Content-Type: application/json
* Http EndPoint: /AutoComplete  (https://api.ezshuttle.co.za/ezx/locationfinder/api/)
QueryString Parameters:

querytext: (string - Query Text - required)
sessionid: (string - Unique Session Identifier for Every Lookup - required)
partnerKey: (string - Will be supplied on account creation -required)

Response Json Body

`{"displayName":"Sandton City, Rivonia Road, Sandhurst, Sandton, South Africa","googlePlaceId":"ChIJu6K6ei1zlR4RS3iNn8RaO24"}`

Once the Google PlaceId is obtained it needs to be converted to an EzShuttle PlaceId:

GooglePlaceIdLookup Service

Http Verb: GET
Headers: Content-Type: application/json
Http EndPoint: /GooglePlaceIdLookup  (https://api.ezshuttle.co.za/ezx/locationfinder/api/)
QueryString Parameters:

googlePlaceId: (string - Google PlaceId Obtained from AutoComplete Service - required)
sessionId: (string - Unique Session Identifier, same as used for AutoComplete Session - required)

Response Body (EzPid - EzShuttle PlaceId)

EZKnYwMDEqMnwtMjYuMTY1NTA4Nzk5OTk5OTl8MjguMTQwNzkxMnwtMXw0OXw3OSBCb2VpbmcgUmQgRSwgQmVkZm9yZHZpZXcsIEdlcm1pc3RvbiwgMjAwNywgU291dGggQWZyaWNhfDEwMDR8MXwxMC4yNzExfA==


# 4 QuickQuotes

Request

* Http Verb: GET
* Headers: Content-Type: application/json
* Http EndPoint: /QuickBookings
* QueryString Parameters:

* pickupDateTime: 2019-07-29T07:00:00  (datetime - required)
* pickupPlaceId: ChIJwymiBTgUlR4R1iEoeUAcv7M  (string - Google PID OR Ezshuttle EzPID - required)
* destinationPlaceId: ChIJ3XLuZMcPlR4RXSWvBLcK5o8  (string - Google PID OR Ezshuttle EzPID - required)
* vehicleType: 1  (int - TypeCode - required)
* numberOfPassengers: 1  (int - Pax, min=1;max=13 - required)
* includeBabySeat: false  (bool - BabySeat is requested - required)
* includeTrailer: false (bool - Trailer is requested - required)
* returnPickupDateTime: 2019-08-19T08:00:00.000Z (datetime - If NULL reservation is one way only - optional)

Response Body 

500000

Response is the quoted amount in ZAR cents. Above quote is therfore R 500.00

Recomended Type Codes 
1 (3 pax) - sedan 
3 (9 pax) - minibus 9 pax
4 (13 pax) - minibus 13 pax

Types can be pulled dynamically from: https://api.ezshuttle.co.za/ezx/quote/api/VehicleType
The results include vehicle thumbnails.

# 5 QuickBookings (Create)

Http Verb: POST
Http EndPoint: /QuickBookings
Headers: Content-Type: application/json
Json Body Content:
```json
`{
  "PickupPlaceId": "ChIJwymiBTgUlR4R1iEoeUAcv7M",       // (string - Google PID OR Ezshuttle EzPID - required) 
  "DestinationPlaceId": "ChIJ3XLuZMcPlR4RXSWvBLcK5o8",  // (string - Google PID OR Ezshuttle EzPID - required) 
  "PickupDisplayAddress": "",                           // (string - Additional address description to be appended to Google/LocatioFinder address - optional) 
  "DestinationDisplayAdadress": "entrance 4",           // (string - Additional address description to be appended to Google/LocatioFinder address - optional) 
  "PickupFlightNumber": "FLTEST",                       // (string - Compulsory if pickup is an airport - optional, on condition) 
  "ReturnPickupFlightNumber": null,                     // (string - Compulsory if pickup is an airport on return trip - optional, on condition) 
  "PickupDateTime": "2019-07-28T07:00:00",              // (datetime UTC - required) 
  "ReturnPickupDateTime": null,                         // (datetime UTC - If NULL reservation is one way only - optional) 
  "NumberOfPassengers": 1,                              // (int - Pax, min=1;max=13 - required) 
  "VehicleType": 1,                                     // (int - TypeCode - required) 
  "IncludeBabySeat": false,                             // (bool - BabySeat is requested - required) 
  "IncludeTrailer": false,                              // (bool - Trailer is requested - required) 
  "SpecialInstructions": null,                          // (string - Special Instructions - optional) 
  "PaymentMethod": 4,                                   // (int - TypeCode 4 = Account - required) 
  "Name": "Richard",                                    // (string - Passenger FirstName - required) 
  "Surname": "McIntyre",                                // (string - Passenger Surname - required) 
  "Cell": "+27722939392",                               // (string - Passenger Mobile - required) 
  "Email": "richard@mailinator.com",                    // (string - Passenger Email - required)
  "PurchaseOrder": "POTEST",                            // (string - Client Defined PurchaseOrder - optional)
  "CostCentre": "COTEST",                               // (string - Special Instructions - optional) 
  "ClientReservationId": "PR884526"                     // (string - Client Defined System Reference - optional) 
}` 
```
Response
```json
`{
    "ReferenceId": 884526,                                  // (int - EzShuttle ReferenceId)
    "PickupPlaceId": "ChIJwymiBTgUlR4R1iEoeUAcv7M",
    "DestinationPlaceId": "ChIJ3XLuZMcPlR4RXSWvBLcK5o8",
    "PickupDisplayAddress": "",
    "DestinationDisplayAddress": "entrance 4",
    "PickupFlightNumber": "FLTEST",
    "ReturnPickupFlightNumber": "",
    "PickupDateTime": "2019-07-28T07:00:00",
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
}`
```
# 6 QuickBookings (Cancel)
* Http Verb: GET
* Headers: Content-Type: application/json
* Http EndPoint: /QuickBookings
* QueryString Parameters:

* reference: (int - EzShuttle Reference - optional)

Response

Http 200-OK
Reservation Has Been Deleted For Id = 123456


# 7 UpdatePurchaseOrderRequest 

Http Verb: POST
Http EndPoint: /UpdatePurchaseOrderRequest
Headers: Content-Type: application/json
Json Body Content:
`{
    ReferenceId: 884526
    PurchaseOrder: "My New PO"
}`

Response

True

# 8 Error Codes 

In the case of an error an HTTP 400 error code will be returned wth the error detail in the body. For example:

`{
    "message": "QuickBookings => POST => Result = Error Occurred",
    "stackTrace": "",
    "errorCode": 83
}`

Please examine the errorCode field to obtain the ezshuttle api error as per table below:


# Error: Validation

 * SMSOrEmailNotificationFailure = 46,
 * BlockTripLuxuryVehicle = 47,
 * BlockTripOver3Passengers = 48,
 * BlockTripUnder4Passengers = 49,
 * BlockTrip = 50,
 * DestinationDestionationTrip = 51,
 * VehicleTypeUnknown = 52,
 * Trip24Hours = 53,
 * NoDropOff = 54,
 * NoPickup = 55,
 * NoPassenger = 56,
 * ParentClientInvalid = 57,
 * ParentClientBooking = 58,
 * InvalidClientPassenger = 59,
 * PassengerNotFound = 60,
 * ClientNotFound = 61,
 * ReservationIdNotNull = 62,
 * ReservationIsNull = 63,
 * ReservationClientNoLongerExist = 64,
 * ReservationNotFound= 65,
 * ReservationIdNullOrZero = 66,
 * NotificationTypeNotSpecified = 67,
 * QuoteConvertedAlready = 68,
 * TripTimeInvalid = 69,
 * PickupTimeIsNull = 70,
 * SuburbsOfficesInvalid = 71,
 * PricePlanNotFound = 72,
 * DestinationInvalid = 73,
 * SuburbInvalid = 74,
 * SuburbDestinationOfficesInvalid = 75,
 * NoAddressSpecified = 76,
 * UnauthorizedVehicleType = 77,
 * CancellationDeadlineExpired=78,
 * AdhocTripsInternalError=79,
 * CannotCancelPaidTrip = 80,
 * ReservationPaymentNotCreditCard = 81,
 * TripOutOfRangeOfOffice = 82,
 * TripOnlineDeadline = 83,
 * JhbPtaSuburbToSuburbNotSupported = 84,
 * SuburbOrDestinationNotFound = 85,
 * TrailerRequiresAdditionalPax = 86,
 * GeneralValidationError = 98,
 * MultiTripReservationNotSupported = 300

# Error: General

* GeneralError = 99,
* Unauthorized = 100,
* InternalServerError = 101,
* ResourceNotFound = 102,
* AccountBlocked = 1000

# Error: Process

* ErrorSendingSMSOrEmailNotification = 200,
* ErrorProcessingPayment = 300

