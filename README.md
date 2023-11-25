# EzShuttle QuickBookings API v2.0

Sections

1. Base API Url
2. Authorization
3. LocationFinder
4. QuickQuotes
5. QuickMultiQuotes
6. QuickBookings (Create)
7. QuickBookings (Cancel)
8. QuickBookings (Get)
9. QuickConfirmations (Get)
10. UpdatePurchaseOrderRequest
11. QuickClientReferenceLookup
12. Error Codes
13. Webhook

# 1 Base API Url

All endpoints described in this document with the exception of LocationFinder have the following base API url:
https://api.ezshuttle.co.za/ezx/quickbooking/api/

# 2 Authorization

API authorization is implemented with HTTP Basic Authentication in order to retain compatibility with certain legacy partner applications.
The username/password is the same as you would use to access the Client Portal: https://ezshuttle.co.za/bookings/ClientPortal/admin.shtml
Should you not have an account please apply for one by clicking "Want To Open An Account" on the above page.

# 3 LocationFinder

Our endpoints accept either Google PlaceId's (Google PID - Google Places AutoComplete) or EzShuttle PlaceId's (EzPID - EzShuttle API Place Finder) 
We recommend the use of EzPID's with our place finder endpoint for the following reasons:

a. Automatically filtered to only provide locations which we service.
b. Built in Handling of IATA codes for airport pickup's / drop offs.
c. Lower latency for Quote Generation / Reservation Creation.

It is recommended that a library such as https://github.com/TarekRaafat/autoComplete.js is used for the Location Finder.
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

* Http Verb: GET
* Headers: Content-Type: application/json
* Http EndPoint: /GooglePlaceIdLookup  (https://api.ezshuttle.co.za/ezx/locationfinder/api/)
* QueryString Parameters:

googlePlaceId: (string - Google PlaceId Obtained from AutoComplete Service OR IATA - required) </br>
sessionId: (string - Unique Session Identifier, same as used for AutoComplete Session - required)

IATA Format (Special case for airports - to be passed in as googlePlaceId parameter. </br>
`|IATA|*** where *** is the IATA code e.g |IATA|PLZ`

Response Body (EzPid - EzShuttle PlaceId)

EZKnYwMDEqMnwtMjYuMTY1NTA4Nzk5OTk5OTl8MjguMTQwNzkxMnwtMXw0OXw3OSBCb2VpbmcgUmQgRSwgQmVkZm9yZHZpZXcsIEdlcm1pc3RvbiwgMjAwNywgU291dGggQWZyaWNhfDEwMDR8MXwxMC4yNzExfA==

# 4 QuickQuotes

Request

* Http Verb: GET
* Headers: Content-Type: application/json
* Http EndPoint: /QuickQuotes/get
* QueryString Parameters:

* pickupDateTime: 2019-07-29T07:00:00 or 2019-07-29T05:00:00Z (SAST DateTime / UTC DateTime - required.)
* pickupPlaceId: ChIJwymiBTgUlR4R1iEoeUAcv7M  (string - Google PID OR EzShuttle EzPID - required)
* destinationPlaceId: ChIJ3XLuZMcPlR4RXSWvBLcK5o8  (string - Google PID OR EzShuttle EzPID - required)
* vehicleType: 1  (int - TypeCode - required)
* numberOfPassengers: 1  (int - Pax, min=1;max=13 - required)
* includeBabySeat: false  (bool - BabySeat is requested - required)
* includeTrailer: false (bool - Trailer is requested - required)
* returnPickupDateTime: 2019-08-19T08:00:00.000Z (datetime - If NULL reservation is one way only - optional)

* Response Headers: Content-Type: text/plain

Response Body 

500000

Response is the quoted amount in ZAR cents. Above quote is therefore R 500.00

Recommended Type Codes 
1 (3 pax) - sedan 
3 (9 pax) - minibus 9 pax
4 (13 pax) - minibus 13 pax

Types can be pulled dynamically from: https://api.ezshuttle.co.za/ezx/quote/api/VehicleType
The results include vehicle thumbnails.

# 5 QuickMultiQuotes

Request

* Http Verb: GET
* Headers: Content-Type: application/json
* Http EndPoint: /QuickMultiQuotes/get
* QueryString Parameters:

* pickupDateTime: 2019-07-29T07:00:00  (UTC DateTime - required. Any TimeZone/Offset information passed will be stripped from request)
* pickupPlaceId: ChIJwymiBTgUlR4R1iEoeUAcv7M  (string - Google PID OR EzShuttle EzPID - required)
* destinationPlaceId: ChIJ3XLuZMcPlR4RXSWvBLcK5o8  (string - Google PID OR EzShuttle EzPID - required)
* numberOfPassengers: 1  (int - Pax, min=1;max=13 - required)
* includeBabySeat: false  (bool - BabySeat is requested - required)
* includeTrailer: false (bool - Trailer is requested - required)
* returnPickupDateTime: 2019-08-19T08:00:00.000Z (datetime - If NULL reservation is one way only - optional)
* includeAvailabilityCheck: true (if true then an availability check will be conducted. Default is false if not specified)

Response Body 
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
    },
    {
        "PrimaryTripQuoteId": "b4dec742-2ffd-499c-8f0b-0a2ab63a32d3",
        "ReturnTripQuoteId": "291bb893-e084-4fd0-a0ec-b0b915a9bca5",
        "TotalAmountInCents": 105000,
        "VehicleTypeId": 2,
        "VehicleTypeName": "Business - Sedan (3 seater)",
        "TermsAndConditionsUrl": "https://www.ezshuttle.co.za/faq/",
        "CancelAllowedDeadlineMinutes": 30,
        "CancelBillableDeadlineMinutes": 120,
        "IsAvailableForOnlinebookingPrimary": true,
        "IsAvailableForOnlinebookingReturn": true
    },
    {
        "PrimaryTripQuoteId": "ed0cee4e-dd9c-4604-a11c-88845310c227",
        "ReturnTripQuoteId": "d37b3d87-1e33-4063-85df-1a2354e7e2d5",
        "TotalAmountInCents": 106000,
        "VehicleTypeId": 3,
        "VehicleTypeName": "Comfort - Van (9 seater)",
        "TermsAndConditionsUrl": "https://www.ezshuttle.co.za/faq/",
        "CancelAllowedDeadlineMinutes": 30,
        "CancelBillableDeadlineMinutes": 120,
        "IsAvailableForOnlinebookingPrimary": true,
        "IsAvailableForOnlinebookingReturn": true
    },
    {
        "PrimaryTripQuoteId": "1110f22d-4a7f-4e30-b2b9-5975aed48c0d",
        "ReturnTripQuoteId": "90996938-303c-4398-ad53-154ac2b308ce",
        "TotalAmountInCents": 106000,
        "VehicleTypeId": 4,
        "VehicleTypeName": "Comfort - Minibus (13 seater)",
        "TermsAndConditionsUrl": "https://www.ezshuttle.co.za/faq/",
        "CancelAllowedDeadlineMinutes": 30,
        "CancelBillableDeadlineMinutes": 120,
        "IsAvailableForOnlinebookingPrimary": true,
        "IsAvailableForOnlinebookingReturn": true
    }
]
```

# 6 QuickBookings (Create)

Http Verb: POST
Http EndPoint: /QuickBookings/post
Headers: Content-Type: application/json
Json Body Content:
```json
{
  "PickupPlaceId": "ChIJwymiBTgUlR4R1iEoeUAcv7M",       // (string - Google PID OR Ezshuttle EzPID - required) 
  "DestinationPlaceId": "ChIJ3XLuZMcPlR4RXSWvBLcK5o8",  // (string - Google PID OR Ezshuttle EzPID - required) 
  "PickupDisplayAddress": "",                           // (string - Additional address description to be appended to Google/LocatioFinder address - optional) 
  "DestinationDisplayAdadress": "entrance 4",           // (string - Additional address description to be appended to Google/LocatioFinder address - optional) 
  "PickupFlightNumber": "FLTEST",                       // (string - Compulsory if pickup/dropoff is an airport - optional, on condition) 
  "ReturnPickupFlightNumber": null,                     // (string - Compulsory if pickup/dropoff is an airport on return trip - optional, on condition) 
  "PickupDateTime": "2019-07-28T07:00:00",              // (UTC DateTime - required.Timezone/Offset will be stripped from                                                             // request) e.g for 9:00AM localtime pickup : 2019-07-28T07:00:00
  "ReturnPickupDateTime": null,                         // (datetime UTC - If NULL reservation is one way only - optional) 
  "NumberOfPassengers": 1,                              // (int - Pax, min=1;max=13 - required) 
  "VehicleType": 1,                                     // (int - TypeCode - required) 
  "IncludeBabySeat": false,                             // (bool - BabySeat is requested - required) 
  "IncludeTrailer": false,                              // (bool - Trailer is requested - required) 
  "SpecialInstructions": null,                          // (string - Special Instructions - optional) 
  "PaymentMethod": "4",                                 // (string - TypeCode 4 = Account - required) 
  "Name": "Richard",                                    // (string - Passenger FirstName - required) 
  "Surname": "McIntyre",                                // (string - Passenger Surname - required) 
  "Cell": "+27722939392",                               // (string - Passenger Mobile, single MSISDN in international format - required) 
  "Email": "richard@mailinator.com",                    // (string - Passenger Email, single valid email address - required)
  "PurchaseOrder": "POTEST",                            // (string - Client Defined PurchaseOrder - optional)
  "CostCentre": "COTEST",                               // (string - Special Instructions - optional) 
  "ClientReservationId": "PR884526"                     // (string - Client Defined System Reference - optional but highly recommended. Should be unique) 
}
```
Response
```json
{
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
}
```
# 7 QuickBookings (Cancel)
* Http Verb: DELETE
* Headers: Content-Type: application/json
* Http EndPoint: /QuickBookings/delete
* QueryString Parameters:

* reference: (int - EzShuttle Reference)

Response

Http 200-OK
Reservation Has Been Deleted For Id = 123456

# 8 QuickBookings (Get)
* Http Verb: GET
* Headers: Content-Type: application/json
* Http EndPoint: /QuickBookings/get
* QueryString Parameters:

* reference: (int - EzShuttle Reference)

Response

```json
{
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
}
```
# 9 QuickConfirmations 

* Http Verb: GET
* Http EndPoint: /QuickConfirmations/get?reference={reference}
* Response Headers: Content-Type: application/pdf

Response

PDF-FILE 

# 10 UpdatePurchaseOrderRequest 

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

# 11 QuickClientReferenceLookup

* Http Verb: GET
* Http EndPoint: /QuickClientReferenceLookup/get/{clientReservationId} 
* Response Headers: Content-Type: text/plain

Response Body 

xxxxxx

Where xxxxxx is the booking ReferenceId (int)
ClientReservationId should match the client defined reference passed into QuickBookings-Create.

If not found an HTTP-404 response will be returned.

It is highly recommended that the client application call this endpoint should a timeout occur during the booking process in order to confirm if a reservation was  successfully created. 

It is the responsibility of the client application to ensure that the ClientReservationId provided during the booking process is unique, as this endpoint will simply return the first booking it finds with the provided ClientReservationId.

# 12 Error Codes 

In the case of an error an HTTP 400 error code will be returned wth the error detail in the body. For example:

`{
    "message": "QuickBookings => POST => Result = Error Occurred",
    "stackTrace": "",
    "errorCode": 83
}`

Please examine the errorCode field to obtain the ezShuttle api error as per table below:


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
 * QuoteConvertedAlreadyOrNotFound = 68,
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
 * QuoteNotAvailable = 94,
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

# 13 Webhook

It is possible register a webhook so that EzShuttle will push updates to trips booked under your account in real-time. Please contact your account manager and provide them with a url that your environment exposes to be used for this purpose. e.g https://www.mytravelcorp.com/api/ez_webhook

Webhook Message Json

```json
{
    "PNR": "BT7Z5YR0",
    "UpdateTypeId": 1, 
    "PickupDT": "2019-11-07T17:00:00Z",
    "PickupDisplay": "Cape Town International Airport",
    "DropOffDisplay": "aha Harbour Bridge Hotel & Suites",
    "PriceInCents": 330000,
    "DriverName": "Tom Smith",
    "DriverEta": "2019-11-07T16:48:00Z",
    "DriverPositionLat": -33.810322,
    "DriverPositionLon": 18.497749,
    "IsCancelled": false,
    "MessageCreateDT": "2019-11-07T16:48:00Z"
}
```

Supported update-type codes

Update Type Code (UpdateTypeId)

1=Create    
2=PickupTimeChanged    
3=PickupDropOffLocationChanged    
4=DriverNameChanged    
5=DriverEtaChanged    
6=DriverLocationChanged   
9=Cancelled    
10=Driver5MinuteArrivalAlarm   
11=TripEndEtaChanged    

Update Type Code Description (Types currently supported)

4. DriverNameChanged :
   A driver has been assigned to this trip. This notification will occur no earlier than 3 hours before the trip.

6. DriverLocationChanged :

    A trip which is currently in progress (Driver is enroute to pickup point OR enroute to drop off point) 

8. Driver5MinuteArrivalAlarm :
   Driver is less tha 5 minutes from the pickup point
