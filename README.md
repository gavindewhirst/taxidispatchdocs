# __Taxi Dispatch API Guidebook__
 
Dive right into it with the [Quickstart Guide][8], it has everything you’ll need to get cracking including information on access to our test environments and Postman scripts to get you up and running straight away. We built our Dispatch API because partners told us they wanted to connect their dispatch and data systems to us directly to manage the bookings, as well as having access to the Partner Portal and the tools available there. 
 
A number of widely used dispatching systems are already connected to Booking.com. Check [to see if your dispatching partner is already part of our [Connectivity program][8]. 
 
To deliver a great experience to our customers we expect the following be completed in all integrations : 
 
* Booking Administration - acknowledging new, updated and cancelled bookings. For reference see :  
    * [New Booking is Available][1]
    * [Sending a Booking Response][1]
    * [Updated Bookings][1]
    * [Cancelled Bookings][1]

* Driver Details - telling us when you have assigned the driver who will perform the ride. See : 
    * [Assignment of Driver][1]

* Journey Progress - if you do not use our driver app you must send us driver events from your own driver app. See : 
    * [Ride Progress][1]

Partners can extend their integrations with the following features : 
* Price and availability requests
* Driver location tracking (required for preferred partners and on-demand ridehail)*
* Update and create pricing campaigns*
* Managing coverage* 
* Receiving and responding to feedback*

# __Quick Start Guide__
Get started quickly and simply. Use Taxi Dispatch API to get access to your bookings and begin connecting your dispatch or information systems : 
 
1. Request an API key for the Sandbox
2. Import our Postman scripts and test with our Sandbox environment
3. Perform your testing against the Sandbox
4. Request access to the LIVE system
 
## 1. __Request an API key for the Sandbox__
If you are a current Partner with Booking.com Taxi then reach out to your account manager and request API keys to access the Sandbox environment. 
 
## 2. __Import Postman scripts__
We have a Postman collection which covers all the main actions available through the Taxi Dispatch API, you can find the collection here. Postman is a cool tool that also allows you to export code into a number of supported languages, this will help you get started really quickly. If you get stuck on an action, get help with these scripts. 
 
Our postman collection provides the following  : 
 
* Example of how to get AUTH tokens
* Example of /bookings call (all future bookings)
* Example of getting a specific booking
* Example of a accept/reject a booking
* Example of a driver assignment
* Example of submitting a driver event

The collection can be found [here][1]. 
 
 
## 3. __Perform testing against the sandbox environment__
The sandbox is an interactive environment that allows you to work with bookings in the same way that you would do in the LIVE system and it’s expected that you will do all your testing in sandbox, there is no testing available in our LIVE system. There are a few points to consider :  
* Sandbox contains TEST BOOKINGS - this will not contain your bookings, nor is it likely to contain bookings for your specific areas of operation. 
* The sandbox data is generated for each AUTH token and will retain the state of the bookings for the duration of the token. 
* This means that any changes you make to the sandbox are dependent on you using the same AUTH token for each testing session
* Pagination is not currently supported in the Sandbox environment (setting a page size will not affect the bookings displayed)
 
## 4. __Request access to the LIVE system__
In our partner portal you should have access to a “CONFIGURATION” section - within here you will be able to generate API keys. 
 
__NOTE : KEEP YOUR KEYS SAFE!!__

The Client ID and Client Secret are both needed to confirm your application’s identity and it is critical that you do not expose or share them. Follow these suggestions to keep the secret safe:
* Do not expose in files such as JavaScript or HTML files in client-side code
* Do not store it in files on a web server that may be viewed externally. For example, configuration files, include files, etc
* Do not store it in log files or error messages
* Do not email it or post it on a message board or other public forums
 
Remember that when exchanging an OAuth 2.0 authorization code for an access token, the Client Secret is passed as part of the request. Make sure you do not expose this request publicly!

## __Authentication Example__
The Taxi Dispatch API endpoints are secured using the OAuth 2.0 standards, using the client credential flow. Your API client ID and secret can be used to retrieve a token which is then used to authenticate requests. These tokens are short-lived and should be updated from time to time by calling the authentication service frequently. 
 
### Example : Fetching a token 
To fetch a token from the Sandbox endpoint you can use the following curl script :

~~~~ 
curl -X POST --user '<CLIENT_ID>:<CLIENT_SECRET>' -H 'Content-Type: application/x-www-form-urlencoded' 'https://dispatchapi-sandbox-qa.auth.eu-west-1.amazoncognito.com/oauth2/token?grant_type=client_credentials' 
~~~~

Replacing the <CLIENT_ID> and <CLIENT_SECRET> with the details that have received will allow you to request a key. The response should look like this : 

~~~~
{
"access_token" : "",
"expires_in" : 3600,
"token_type" : "Bearer"
}
~~~~
 
You can find the token to use in the property access_token. 
 
Example : Using the token
Using the /booking endpoint to get a basic list of rides : 
 
~~~~
curl -X GET -H 'Content-Type: application/json' -H 'Authorization: <ACCESS_TOKEN>' https://dispatch-api-sandbox.qa.someonedrive.me/v1/bookings
~~~~

Replacing the <ACCESS_TOKEN> with the access_token fetched will allow you to list all the bookings available to that token in the sandbox. 

# Booking Status Changes
The below table describes all the possible states of a booking and the status after a /responses call has been made. 

|   | __ACCEPT__ | __REJECT__ |
|---|:---:|:---:|
| __NEW__ | ACCEPT | REJECT |
| __ACCEPTED__ | ACCEPTED | CANCELLED |
| __DRIVER_ASSIGNED__ | ACCEPTED* | CANCELLED |
| __PENDING_AMENDMENT__ | ACCEPTED* | CANCELLED |
| __PENDING_CANCELLATION__ | ACCEPTED or DRIVER_ASSIGNED| CANCELLED** |
| __CANCELLED__ | CANCELLED** | CANCELLED** |

\*Sending multiple ACCEPT responses is allowed but the status is unchanged

\**These responses are not expected and we will return an error if attempted

# __Working with Polling__
You are able to get a list of all your future bookings using the /bookings endpoint. Regularly checking this endpoint will allow you to monitor any new bookings received and is an alternative to using our webhooks. 
 
When polling we recommend that you poll at an interval no shorter than __5 minutes__. Rate-limit measures will be put in place for any partner persistently using shorter intervals. If you need to ensure you know about bookings in real-time (i.e. on-demand bookings) then using Webhooks is more appropriate. 
 
When checking using Polling, please be aware that the AUTH key has a lifetime of 1 hour. 
 
# __Working with Webhook Notifications__

You can subscribe to webhooks from within the Partner Portal administration page.

## __Booking Notifications__
/v2/bookings describe webhook notifications, an alternative to polling the /bookings endpoints and can be used to trigger an action within your app or platform. Using webhooks will mean you get these notifications in real-time and have to make fewer API calls to us.
 
The /bookings endpoint is the source of truth and should be used to understand the latest booking information. state_hash is an important concept when working with responses and ensures that we know you are always working with the most current information. 

### __Important Note on Booking Notifications__ 
We issue webhooks as notifications and therefore we do not require your immediate action so we are not awaiting your response. <span style="color: red">If your endpoints are down, we will not currently attempt a re-try. You may get multiple notifications for the same change. You may not get the notification.</span> All these things can happen and you should handle these situations correctly. 
 

# __Booking Administration__
 Everything you need to manage bookings is available via /v1/bookings 
 
* New Bookings
* Updated Bookings
* Cancelled Bookings
 
All of the above actions require a response. To process the above, please see : [Sending a Booking Response][2]

## __New Bookings__
The customer has committed to purchasing the ride with you. Once this has been processed by our fraud teams we make this booking available to you on the terms of your commercial contract.  

* If subscribed to /v2/booking (webhook notifications) you will receive a notification on the subscribed endpoint. See [Working with Webhooks][3].
* The booking will also be available via the /bookings endpoint and will be of status NEW. 
* The booking will remain in this status until ACCEPTED or REJECTED using the responses endpoint (/bookings/{reference}/{legid}/responses)


Updates received after being accepted will push the booking into a PENDING_AMENDMENT status
If the booking is cancelled before the booking is ACCEPTED, the booking will be pushed to PENDING_CANCELLATION
We expect all notifications and booking updates to be processed in a timely manner and appropriate responses received.  
 
Please see : [Sending a Booking Response][2], [Booking Status Changes][4]

## __Updated Bookings__

If a booking is made, but subsequently changes (i.e. a flight number is provided by the customer) then we expect to see a confirmation from partners that this change has been processed. 

* If subscribed to /v2/booking (notifications) you will receive a notification on the subscribed endpoint informing you that the booking has been updated. 
* The booking will also be available via the /bookings endpoint and will be of status PENDING_AMENDMENT unless NEW[^1]
* The booking will remain in this status until ACCEPTED or REJECTED using the responses endpoint (/bookings/{reference}/{legid}/responses)

If the booking is cancelled before the response is received the booking will be pushed to PENDING_CANCELLATION
 
__We expect all notifications and booking updates to be processed in a timely manner and appropriate responses received.__
 
Please see : [Sending a Booking Response][2], [Booking Status Changes][4] 

## __Cancelled Bookings__
If a booking is made and is subsequently cancelled by the customer we expect to see a confirmation from partners that this change has been seen and processed. Cancelled rides cannot be "rejected", the ride has been cancelled.  

* If subscribed to /v2/booking (notifications) you will receive a notification on the subscribed endpoint informing you that the booking has been cancelled. 
* The booking will also be available via the /bookings endpoint and will be of status PENDING_CANCELLATION.
* The booking will remain in this status until an ACCEPT response is received, after which it will move to CANCELLED.

__The only valid response is ACCEPT as this booking has been cancelled and cannot be recovered.__ 

Please see : [Sending a Booking Response][2], [Booking Status Changes][4]



## __Sending a Booking Response__  
Once a customer makes a booking, amendment or cancellation we expect confirmation from our partners that this has been seen/acknowledged. 

Booking acknowledgements are made using : /bookings/{reference}/{legid}/responses
 
 The responses endpoint has the following parameters : 
 
* __supplierResponse__ - should be one of the following : 
  * ACCEPT
    * Acknowledges any changes that are pending.
  * REJECT
    * Rejects the action and cancels the booking
* __state_hash__ - the state hash of the booking we are updating
* __cancellationReason__ - (optional) should be one of the following : 
    * CANT_FULFILL_CUSTOMER_REQUEST
    * INCOMPLETE_BOOKING_DETAILS
    * NO_AVAILABILITY
    * RATE_ERROR
    * ROAD_CLOSURE
    * OTHER

NOTE : For a full understanding of what a Response will do to booking status, please see [Booking Status Changes][4].  
Full documentation for accepting changes can be found [here][3].  

## __Assignment of Driver__
The driver assignment will take driver details (first_name, last_name, telephone_number) and associate these against the booking. Customer contact details are available in the /booking responses. 

Drivers need not be registered beforehand and can be assigned freely if you are using your own driver app. If you intend to use the Booking.com Driver app, you should include the driverID property. Registering a driver on the /v1/drivers endpoints will send an activation link to the phone and enroll driver app use. See Driver Management. 

Assignment of a driver also acts as an implicit ACCEPT for NEW bookings.

Documentation for assigning drivers can be found in our [Apiary documentation][6].  


## __Ride Progress__ 
Once a driver has been assigned, and the pickup time is near, we expect to understand the status and location of the driver/car, so we are able to offer the best understanding to our customer and customer support teams. 
 
We ask that you use our driver app, or send us events from your own app. If you wish to use our driver app please see the Managing Drivers section. 
 
We support receipt of the following event types : 
 
* DRIVER_DEPARTED_TO_PICKUP
    * The driver has set off and is en-route to the pickup location
* DRIVER_ARRIVED_AT_PICKUP 
    * The driver has arrived at the agreed pickup location and is awaiting the customer
* DRIVER_SUBMITTED_CUSTOMER_NO_SHOW
    * The customer hasn’t arrived in the allotted time and the driver is leaving the pickup location
* DRIVER_DEPARTED_TO_DROPOFF
    * Customer is in the car and the ride has begun
* DRIVER_ARRIVED_AT_DROPOFF
    * Customer has exited the car at the drop-off location
 
Expected Flow : #image here#
 
Documentation for submitting ride progress can be found here -> 
https://taxidispatchapi.docs.apiary.io/#reference/0/v1bookingsreferencelegiddriverevents/push-driver-events-for-the-booking
 
Example:  Driver event being submitted : 

~~~~
curl --location --request POST 'https://dispatch-api-sandbox.qa.someonedrive.me/v1/bookings/50989929/49453100/driver/events' \
--header 'Authorization: Bearer <AUTH_TOKEN>' \
--data-raw '{ 
    "event_type": "DRIVER_DEPARTED_TO_PICKUP", 
    "latitude": 53.4883114, 
    "longitude": -2.2522131, 
    "occurred_at": "2020-09-22T12:05:01Z"
}'
~~~~

# __Price and availability requests__
At this point, our customers are searching for partners who are able to provide them with the right ride - they tell us their pickup and drop-off locations, the date and time at which they want to travel and the number of passengers that will be travelling with. We search all our partners for the best rates available and return a selection of car types for them to choose from. 
 
There are three ways we will try and calculate a price for a ride :
 
* By using the rate information you provide to us - you can do this using our partner portal. We allow you to provide pricing models based on distance travelled and prices based on fixed routes.__link to portal help__
 
* By uploading to us your rates using the /rates-upload endpoints
 
* By subscribing to the /rates endpoint - each time we receive a request for a price we will send you a request containing the information required to price.
  
## __Subscribing to the /rates endpoint__
You can setup the price subscriptions within our Partner Portal API administration page. <link>.

Your /rates endpoint should be configured with OAuth 2.0 authentication or a long lived token that you provide to us. 

We will ask you for : 
* /rates endpoint - the endpoint at which you will receive the rates requests
* Authentication - Your long lived security token
 
 ## __Important note on /rates volumes__
<span style="color:red">When a price request is received we will forward our request to the endpoint you configured within the Partner portal. Depending on your coverage this can generate a volume of price requests. Please speak with your account manager to discuss expectations.
 
__If quotes are not received within 2000ms of being requested, your response will not be considered as part of our selection process.__
</span>

Diagram of expected flow : 


Request payload for the /rates endpoint can be in our [API spec][7]. 






[1]: https://www.getpostman.com/collections/329c525522555a7a74cd
[2]: #booking_response 
[3]: #working_with_webhooks
[4]: #booking_status_changes
[5]: https://taxidispatchapi.docs.apiary.io/#reference/0/v1bookingsreferencelegidresponses/provide-an-accept-or-reject-response-for-a-booking-in-its-current-state
[6]: https://taxidispatchapi.docs.apiary.io/#reference/0/v1bookingsreferencelegiddriver/assign-a-driver-to-a-single-taxi-booking
[7]: https://taxidispatchapi.docs.apiary.io/#reference/0/rates/search-different-transport-categories
[8]: #quick_start_guide

