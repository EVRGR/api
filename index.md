---
title: Evergreen REST API
mediaroot: media
markdown2extras: wiki-tables
---

# Evergreen API

This is the description for the REST interface between an application (mobile or web) and the server. Each site can act as a 'device' and have an Evergreen wallet for each of it's users. It can also allow Evergreen users to withdraw from the account to the wallet. This is an easy way of implementing an internal fee-free marketplace.

### All API calls start with

<pre class="base">
{{url_base}}
https://play.evr.gr/rest/v01
https://eu.evr.gr/rest/v01

</pre>

### Format

All responses are **JSON**

# Metadata for all requests:

All requests contain a metadata block that contains information about the user's device,
authentication, software version, OS version, and language.

### Example:

	  "meta":
	  {
       "uuid": "7a2a7841-a8c8-42c8-89e0-8043c7721327",
       "evg_auth_token": "A49289392771D3849F982929",
	   "push_token": "F992829294892918482002948",
	   "platform" : "ios",
	   "platform_version": "6.1.3",
       "device_model": "iPhone4,1",
       "client_version": "0.9",
	   "language": "en_US",
	   "request_id": "3985829askdjk93859dkfj",
	   "lat" : 53.204829,
	   "lon" : 15.29484
	  }

#### Where:

**`uuid`** (*required*) is the device's unique identifier, generated by BPXUUID on iOS.

**`evg_auth_token`** (*required except in login*) is the EVG authentication token to validate the request.  If an auth token has not yet been assigned, this parameter should not be sent.

**`push_token`** (*optional*) is the APNS token, if available for the user's device.

**`platform`** (*optional*) is the device's major platform, either `android` or `ios`

**`platform_version`** (*optional*) is the device's operating system version, as reported by the  device.

**`device_model`** (*optional*) is the user's current device model, for example, "iPhone4,1" for the iPhone 4S.  To get the current device model, [see here]( https://github.com/cyclestreets/ios/blob/master/lib/categories/UIDevice%2BMachine.m)

**`client_version`** (*required*) is the software version of the client as a string.

**`language`** (*optional*) is the BCP 47 language/locale tag appropriate for the client

**`request_id`** (*required*) is a client-generated request ID.  If two requests are received with the same request ID, the action will only be completed once.  If a second request is made with the same ID, the result of the first request should be returned if possible.

**`lat`** (*optional*) is a floating point value, indicating the user's current latitude, + for north, - for south.  Should be present at least on requests on startup and shutdown.

**`lon`** (*optional*) is a floating point value, indicating the user's current longitude, + for east, - for west.  Should be present at least on requests on startup and shutdown.

# Validation
All requests goes through in depth validation of post parameters. In the case of failure it returns status 400 and a body with "path" being the parameter that failed and "msg" basic information about the validation error. The validations in place can be found [in the code](https://github.com/FractasticalLabs/evergreen-core/blob/master/src/evergreen_core/api/schemas.clj), and examples of failing cases can be found [in the tests](https://github.com/FractasticalLabs/evergreen-core/blob/master/test/evergreen_core/api/schemas_test.clj)

# User IDs

Many REST calls take or return a user_id.  This is a string of the format `login_type:login_id`, where `login_type` current lin is either `facebook` or `qr`.  for `qr`, login_id is a string of the form: `https://qr.evr.gr/ID` where `ID` is a 16 character random hex string, assigned by the server.


# Metadata for all responses:

	"meta":
	{
	 "status": "upgrade",
	 "message": "Please upgrade your Evergreen app to the newest version on the iTunes Store to continue using Evergreen.",
	 "next_step": "go_to_url",
	 "url": "https://itunes.apple.com/498r8dsakkjssfa",
	 "wallet_balance": "150",
	 "account_balance": "1000"
	}

#### Where:

**`status`** (*required*) is the request status.  Allowable values are:

- `authorized`: Continue as normal.

- `unauthorized`: Authentication was not valid for this resource.  Display `message`, and proceed with `next_step` and/or `url` if present. 

- `error`: There was a server error.  Display `message` and proceed with `next_step` and/or `url`.  Log the error and report if possible.

- `upgrade`: The client version is too old to be used.  Display `message`, and then proceed with `next_step` (usually `go_to_url`), usually a link to the appropriate app store.

**`message`** (*optional*) A textual message to display to the user in a dialog box immediately if this key is present.  The dialog should be presented with only an OK button, that when pressed performs the action in `next_step` if present.

**`next_step`** (*optional*) The next step to take after the message is presented.  If this key is not present, continue as normal.  Allowable values are:

- `home`: Go to the home screen.
- `logout`: Erase auth token and return to the login screen.
- `go_to_url`: Go to this URL in an external browser immediately.
- `register`: User is not registered on this server. Redirect to registration screen.

**`url`** (*optional*) The URL to go to if `next_step` is `go_to_url`.

**`wallet_balance`** (*optional*) The new wallet balance after this request, if any.  Sent as a string, and should be processed as a decimal number.  For v1 will always be an integral value.

**`account_balance`** (*optional*) The new account balance after this request, if any.  Sent as a string, and should be processed as a decimal number.  For v1 will always be an integral value.

# Account

## POST /register_user

Registers a new user. One must either provide a standard email/password or a facebook login. Can be performed from any mobile device or website or "terminal app" (the special merchant version of the app).

#### example request

    POST {{url_base}}/register_user
    
    {
	  "meta" : {...},
	  "user_id": "facebook:123412124",
      "pin": "12346",
      "name": "James Jenkins",
      "passphrase": "superSecret1",
      "email": "james.jenkins@gmail.com",
      "qr_code": "5v4XR37c9lzp8E1bZIO1L2956S5643"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`user_id`** (*optional*) is the EVG internal user id of the user.  User_id takes the form
`facebook:<facebook_user_id>` or `qr:<qr code>`

**`pin`** (*required*) is secret PIN code of the user. Must be five digits.

**`name`** (*optional*) The name that a user wants to be called. 

**`passphrase`** (*optional*) optional passphrase for a user account. Must contain a lowercase, uppercase, and non-letter character.

**`email`** (*required*) Optional email for a user

**`qr_code`** (*optional*) Associate a qr_code with a user and create associated device. 


#### response

     {
	 "meta": {...},
	 "status":"error",
	 "message":"username can only contain alphanumeric characters",
	 "evg_auth_token": "A49289392771D3849F982929"
   
     }

##### Where:

**`status`** (*required*) The result of the registration.  Valid responses are:

- `success`: User was successfully registered

- `error`: Unspecified error.

**`message`** (*optional*) Message that accompanies error. (i.e. "no user")

**`evg_auth_token`** (*optional*) is a string, the Evergreen auth token to be used with future requests to the server. Should be returned if user registers on a mobile device.

**`email`** (*optional*) Optional email for a user to be used for login

**`passphrase`** (*optional*)  Passphrase for a user account



## POST /register_device

Register a new device (only a device, account must already exist) with the server. One must either provide a standard email/password or facebook. 

#### POST /register_device

Registers a new device for a user.

    POST {{url_base}}/register_device
    
    {
	  "meta" : {...},
      "user_id": "facebook:123412124",
      "facebook_auth_token": "939ae7492900f3c8902093b"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`user_id`** (*required*) is the EVG internal user id of the user.  For now, only facebook login
is supported, and user_id therefore takes the form `facebook:<facebook_user_id>`

**`facebook_auth_token`** (*required*) is the auth token received from facebook.

#### response

     {
	 "meta" {...},
     "evg_auth_token": "A49289392771D3849F982929",
     "name": "Joel Dietz"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`status`** (*required*) is either `authorized` or `unauthorized`

**`evg_auth_token`** (*required*) is a string, the Evergreen auth token to be used with future requests to the server.

**`name`** (*required*) the name of the person associated with the user_id.

## POST /account_info

Gets the user's account information.  This is, for now, contained entirely in the meta block.

#### example request

    POST {{url_base}}/account_info
    
    {
	  "meta" : {...},
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.


#### response

     {
	 "meta": {...},
	 "name": "Charlie Chaplin",
	 "email": "winter.light@gmx.ch",
	 "is_merchant" : "false"
     }


##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`name`** (*required*) The name that a user wants to be called. 

**`email`** (*required*) Email for a user if provided, empty string if not

**`is_merchant`** (*required*) Whether or not the user is registered as a merchant


## POST /register_merchant

Change merchant specific information 

#### example request

    POST {{url_base}}/register_merchant

    {
	  "meta" : {...},
      "is_merchant": "true",
      "description": "I sell cheese. Contact me at cheesy@eiffeltower.fr"
    }

##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`user_id`** (*required*) is the EVG internal user id of the user.  For now, only facebook login
is supported, and user_id therefore takes the form `facebook:<facebook_user_id>`

**`is_merchant`** (*required*) turns the merchant listing on in the map. 

**`description`** (*optional*) Description that will appear next to merchant's name in listings 

#### response

     {
	 "meta": {...},
	 "status":"error",
	 "message":"invalid passphrase"
     }

##### Where:

**`status`** (*required*) The result of the registration .  Valid responses are:

- `success`: Account information was successfully changed

- `error`: Unspecified error.

**`message`** (*optional*) Message that accompanies error. 

# Payment

## POST /suggested_payors
Get a list of suggested payees based on nearby users.  Takes a lat/lon position.  The response contains a recent payment partners and nearby payment partners list.

#### Example request

     POST {{url_base}}/suggested_payors

     {
	  "meta": {...},
	 }

#### Where:

**`meta`** (*required*) is the request metadata block described above

#### Example response

      {
       "meta": { ... },
       "recent":
	   [
	    {
		 "name": "Dave Kammeyer",
		 "user_id": "facebook:1234121232",
		 "image": "https://s3.aws.com/9583ksajka8493.jpg",
		 "positive_rating": 80,
		 "negative_rating": 0
		
	    },
		{
		 "name": "Moe Burney",
		 "user_id": "facebook:213123213",
		 "image": "https://s3.aws.com/akjsk389kdka.jpg",
		 "positive_rating": 85,
		 "negative_rating": 1
	    }
	   ],
	   "nearby":
	   [
	    {
	    "name" : "Agora Cafe",
	    "user_id": "facebook:2131212345",
        "image": "https://s3.aws.com/4898asijfkskd.jpg",
		"lat": 50.23849284,
		"lon": 15.39482929,
		"last_seen": "2013-03-21T14:20:33Z",
		"distance" : 120,
		"positive_rating": 95,
		"negative_rating": 0
		}
	   ]
      }


**`meta`** (*required*) is the response metadata block described above.

**`recent`** (*required*) is the list of recent payment partners for the user.  This may be an empty list, but the key is required.

**`nearby`** (*required*) is a list of physically nearby users.

**`name`** (*required*) is the name of the suggested user.

**`user_id`** (*required*) is the EVG login ID of the user.  Example format: `facebook:<facebook_user_id>`

**`image`** (*optional*) is a URL to an image file for the suggested
  user.

**`lat`** (*optional*) The potential payor's latitude, for nearby payors

**`lon`** (*optional*) The potential payor's longitude, for nearby payors

**`distance`** (*optional*) distance in meters from user. 

**`last_seen`** (*optional*) When the potential payor's location was last updated,
  for nearby payors

**`positive_rating`** (*optional*) number from 1-100 representing the trustworthiness of a user.

**`negative_rating`** (*optional*) number from 1-100 representing the lack of trustworthiness of a user.


## POST /search_users

#### example request

    POST {{url_base}}/search_users

      {
       "meta": { ... },
	   "query": "Dave"
      }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`query`** (*required*) is user's search query. 

#### response

      {
       "meta": { ... },
       "results":
	   [
	    {
		 "name": "Dave Kammeyer",
		 "user_id": "facebook:1234121232",
		 "image": "https://s3.aws.com/9583ksajka8493.jpg",
		 "positive_rating": 80,
		 "negative_rating": 0

	    },
		{
		 "name": "Dave Johnson",
		 "user_id": "facebook:545443544",
		 "image": "https://s3.aws.com/akjskasdk3992dka.jpg",
		 "positive_rating": 80,
		 "negative_rating": 20
	    }
	   ],
      }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`results`** (*required*) is the list of search results.

**`name`** (*required*) is the name of the user.

**`user_id`** (*required*) is the EVG login ID of the user.  Example format `facebook:<facebook_user_id>`

**`image`** (*optional*) is a URL to an image file for the user.

**`positive_rating`** (*optional*) number from 1-100 representing the trustworthiness of a user.

**`negative_rating`** (*optional*) number from 1-100 representing the lack of trustworthiness of a user.

## POST /make_payment

Make a payment to a user, possibly in response to a payment request.

#### example request

    POST {{url_base}}/make_payment
    
    {
	  "meta" : {...},
	  "transaction_id": "3948839skdka92askdf",
	  "amount": "272",
	  "payee": "facebook:123412124",
	  "request_description": "Because I like you"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`transaction_id`** (*optional*) The transaction id of the request, if the payment is in response to a request.

**`amount`** (*required*) The amount of the transaction, in EVG credits, as a string.  Should be processed as a decimal value, but for now, only integral values of credits are allowed. **Must be positive**

**`payee`** (*required*) The EVG user id to be paid.  Can be of the form: `facebook:<facebook id>` or `qr:<qr code>`

**`request_description`** (*optional*) A description of the
  transaction, provided by the user, shown to the payee.

**`currency_amount`** (*optional*) The corresponding amount of traditional currency, if any.

**`currency_type`** (*optional*) The ISO 4217 currency being loaded onto the card, a string.  Required if `currency_amount` is present.

** item_ids ** (*optional*) An array of items ids as strings (as provided by /items)

#### response

     {
	 "meta": {...},
	 "status": "insufficient_funds",
	 "status_message": "Your account balance is only 200ε, you cannot make this payment."
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`status`** (*required*) The result of the payment request.  Valid responses are:
- `success`: The payment was successful.

- `insufficient_funds`: The Wallet balance is insufficient to make the transaction.

- `invalid_account`: The payee account is invalid.

- `error`: Some other error

**`status_message`** (*optional*) A message to display to the user modally to explain the result of the transaction


## POST /request_payment

#### example request

    POST {{url_base}}/request_payment
    
    {
	  "meta" : {...},
	  "payor": "facebook:1234121232",
	  "amount": "272",
	  "request_description": "Payment for rideshare to Rome."
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`payor`** (*required*) The EVG ID of the user that payment is being requested from

**`amount`** (*required*) The number of EVG credits requested, as a string.  Should be processed as a decimal number, but for now only integral values of credits are allowed.

**`request_description`** (*required*) A description of the transaction, provided by the user, shown to the payee.

** item_ids ** (*optional*) An array of items ids as strings (as provided by /items)

#### response

     {
	 "meta" {...},
	 "transaction_id": "3948839skdka92askdf"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`transaction_id`** (*required*) A unique Transaction ID representing the request and any subsequent payment.  The Client should hold on to this to request status updates. 


## POST /cancel_request

Cancels request that has been made by payee 

#### example request

    POST {{url_base}}/cancel_payment
    
    {
	  "meta" : {...},
	   "transaction_id": "3948839skdka92askdf"
	
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`transaction_id`** (*required*) A unique Transaction ID representing the request.  A person must be the originator of the request in order to cancel it. 

#### response

     {
	 "meta" {...},
	 "status" : "success"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.


## POST /reject_payment

Reject a previous request for payment.

#### example request

    POST {{url_base}}/reject_payment
    
    {
	  "meta" : {...},
	  "transaction_id": "3948839skdka92askdf",
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`transaction_id`** (*required*) The transaction ID if of the request to reject.

#### response

     {
	 "meta": {...},
	 "status": "already_paid",
	 "status_message": "The payment has already been sent!"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`status`** (*required*) The status of the rejection, possible values are:

- `success`: The request has been rejected

- `already_paid`: The request has already been paid (by another device)

- `error`: Some other error.

**`status_message`** (*optional*) A status message to display in an dialog box explaining an exceptional condition.


# ATM Functionality

Withdraw and depositing credits from the user's EVG account.

## POST /withdraw

#### example request

    POST {{url_base}}/withdraw
    
    {
	  "meta" : {...},
	  "amount": "300",
	  "PIN": "12346"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`amount`** (*required*) The number of credits to withdraw.  Handle as decimal, integral only for now.

**`PIN`** (*required*) The user's PIN code.


#### response

     {
	 "meta": {...},
	 "status": "insufficient_funds",
	 "status_message" : "You only have 100 credits available in your account!"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`status`** (*required*) One of the following:

- `success`: The withdrawal was successful.

- `insufficient_funds`: The user does not have enough funds in his bank account to make the withdrawal.

- `wrong_pin`: The wrong PIN code was supplied.

- `error`: There was some other error.

**`status_message`** (*optional*) A message to display in a dialog to the user explaining any exceptional result of the transaction.

## POST /deposit

#### example request

    POST {{url_base}}/deposit
    
    {
	  "meta" : {...},
	  "amount": "34"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`amount`** (*required*) The number of credits to deposit.  Handle as decimal, integral only for now.

#### response

     {
	 "meta": {...},
	 "status": "success"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`status`** (*required*) One of the following:

- `success`: The deposit was successful.

- `error`: Undefined error.

**`status_message`** (*optional*) A message to display in a dialog to the user explaining any exceptional result of the transaction.

## POST /transaction_list

Return a list of recent transactions, paginated.

#### example request

    POST {{url_base}}/transaction_list
    
    {
	  "meta" : {...},
	  "start_position": "0",
	  "num_items": "10"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`start_position`** (*optional*) The starting position in the list of most recent transactions, ordered from most recent, starting at 0.  If `start_position` is omitted, start at position 0 

**`num_items`** (*optional*) Return at most `num_items` items.  If not specified, return all items after `start_position` 

#### example response

     {
	 "meta": {...},
	 "num_transactions": "2",
	 "transactions":
	 [
	  {
       "transaction_id": "3895askjfka838dkf",
	   "transaction_status": "completed",
	   "request_date": "2013-03-21T14:21:33Z"
	   "payment_date": "2013-03-21T18:25:43Z"
	   "amount": "200",
	   "payor_id": "facebook:1234121232",
	   "payor_name": "Dave Kammeyer",
	   "payor_image": "https://s3.aws.com/9583ksajka8493.jpg",
	   "payee_id": "facebook:123412124",
	   "payee_name": "Joel Dietz",
	   "payee_image": "https://s3.aws.com/i388d0d983ks34.jpg",
	   "request_description": "Payment for rideshare to Rome.",
	   "feedback_status": "negative",
	   "feedback_description": "Goods Not Delivered"
      },
	  {
       "transaction_id": "39488akfksadjfk9384838",
	   "transaction_status": "requested",
	   "request_date": "2013-03-22T19:21:33Z"
	   "amount": "200",
	   "payor_id": "facebook:123412124",
	   "payor_name": "Joel Dietz",
	   "payor_image": "https://s3.aws.com/i388d0d983ks34.jpg",
       "payee_id": "facebook:1234121232",
	   "payee_name": "Dave Kammeyer",
	   "payee_image": "https://s3.aws.com/9583ksajka8493.jpg",
	   "request_description": "Give me my money back!",
      }
	 ]

     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`num_transactions`** (*required*) the total number of transactions for the user.  Can be used to decide how many page requests to send.

**`transactions`** (*required*) the list of transactions.

**`transaction_id`** (*required*) is the ID of the transaction.

**`transaction_status`** (*required*) The status of the transaction: Can be:

- `requested`: Money has been requested, but no response has been received yet.

- `rejected`: The request was rejected by the recipient.

- `completed`: The transaction has been completed.

**`request_date`** (*optional*) Is the date of the payment request, if any, in ISO8601 format.

**`payment_date`** (*optional*) Is the date of the payment, if any, in ISO8601 format.

**`amount`** (*required*) is the number of EVG credits of the transaction.  Processed as decimal, integral only for v1

**`payor_id`** (*required*) is the EVG user id of the payor. 

**`payor_name`** (*required*) is the natural name of the payor.

**`payor_image`** (*optional*) is the URL of an image of the payor.

**`payee_id`** (*required*) is the EVG user id of the payee.

**`payee_name`** (*required*) is the natural name of the payee.

**`payee_image`** (*optional*) is the URL of an image of the payee.

**`request_description`** (*optional*) is a description of the payment request, if any.

**`feedback_status`** (*optional*) If the transaction was completed, The buyer's feedback on the transaction  Can be:

- `none`: The buyer has not yet left feedback.

- `positive`: The buyer has left positive feedback.

- `negative`: The buyer has left negative feedback.

**`feedback_description`** (*optional*) If the feedback is negative, a description of the problem with the transaction.  Examples: "Goods not delivered", "Goods not as described", "Goods of lower quality than expected"

## POST /transaction_detail

#### example request

    POST {{url_base}}/transaction_detail
    
    {
	  "meta" : {...},
	  "transaction_id": "3895askjfka838dkf"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`transaction_id`** (*required*) The ID of the transaction to list 

#### response

     {
	 "meta": {...},
     "transaction_id": "3895askjfka838dkf",
	 "transaction_status": "requested",
	 "request_date": "2013-03-21T14:21:33Z"
	 "payment_date": "2013-03-21T18:25:43Z"
	 "amount": "200",
	 "payor_id": "facebook:1234121232",
	 "payor_name": "Dave Kammeyer",
	 "payor_image": "https://s3.aws.com/9583ksajka8493.jpg",
	 "payee_id": "facebook:123412124",
	 "payee_name": "Joel Dietz",
	 "payee_image": "https://s3.aws.com/i388d0d983ks34.jpg",
	 "request_description": "Payment for rideshare to Rome.",
	 "feedback_status": "negative",
	 "feedback_description": "Goods Not Delivered"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`transaction_id`** (*required*) is the ID of the transaction.

**`transaction_status`** (*required*) The status of the transaction: Can be:

- `requested`: Money has been requested, but no response has been received yet.

- `rejected`: The request was rejected by the recipient.

- `completed`: The transaction has been completed.

**`request_date`** (*optional*) Is the date of the payment request, if any, in ISO8601 format.

**`payment_date`** (*optional*) Is the date of the payment, if any, in ISO8601 format.

**`amount`** (*required*) is the number of EVG credits of the transaction.  Processed as decimal, integral only for v1

**`payor_id`** (*required*) is the EVG user id of the payor. 

**`payor_name`** (*required*) is the natural name of the payor.

**`payor_image`** (*optional*) is the URL of an image of the payor.

**`payee_id`** (*required*) is the EVG user id of the payee.

**`payee_name`** (*required*) is the natural name of the payee.

**`payee_image`** (*optional*) is the URL of an image of the payee.

**`request_description`** (*optional*) is a description of the payment request, if any.

**`feedback_status`** (*optional*) If the transaction was completed, The buyer's feedback on the transaction  Can be:

- `none`: The buyer has not yet left feedback.

- `positive`: The buyer has left positive feedback.

- `negative`: The buyer has left negative feedback.

**`feedback_description`** (*optional*) If the feedback is negative, a description of the problem with the transaction.  Examples: "Goods not delivered", "Goods not as described", "Goods of lower quality than expected"


## POST /leave_feedback

#### example request

    POST {{url_base}}/leave_feedback
    
    {
	  "meta" : {...},
	  "transaction_id": "39488akfksadjfk9384838",
	  "feedback_type":"positive"
	
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`transaction_id`** (*required*) is transaction id to leave feedback for.

**`feedback_type`** (*required*) Either

- `positive`: Positive feedback

- `negative`: Negative feedback

**`feedback_description`** (*optional*) If feedback is negative, a description of the problem.

#### response

     {
	 "meta": {...}
    
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.


# Merchant 

Flow for registered merchants or users looking for merchants. Registered merchants can take payments from non-users

## POST /suggested_merchants
Get a list of suggested merchants based on nearby users.  Takes a lat/lon position. Returns 20 closest merchants to passed location. If no location is passed, use user's current location (provided in meta). 

#### Example request

     POST {{url_base}}/suggested_merchants

     {
	  "meta": {...},
	  "lat": 50.238494,
	  "lon": 15.394829,
	  "type": "health"
	 }

#### Where:

**`meta`** (*required*) is the request metadata block described above

**`lat`** (*optional*) The location from which to start searching

**`lon`** (*optional*) The location from which to start searching

**`type`** (*optional*) Type of merchants to search for. Examples include "restaurant," "cafe," "heath," "beauty," "department store," "groceries," etc.  

#### Example response

      {
       "meta": { ... },
       "merchants":
	   [
	    {
	    "name" : "Agora Cafe",
	    "description" : "Cafe/Restaurant, open M-F 12-9pm",
	    "type" : "restaurant",
	    "user_id": "facebook:2131212345",
        "image": "https://s3.aws.com/4898asijfkskd.jpg",
		"lat": 50.23849284,
		"lon": 15.39482929,
		"distance" : 120,
		"positive_rating": 95,
		"negative_rating": 0
		},
	    {
	    "name" : "Kim's Yoga",
	    "description" : "check awesomekimsyoga.com for details",
	    "type" : "health",
	    "user_id": "facebook:2131212323",
        "image": "https://s3.aws.com/4898asijfkskd.jpg",
		"lat": 50.23849211,
		"lon": 15.39482800,
		"distance" : 25000,
		"positive_rating": 80,
		"negative_rating": 1
		}
	   ]
      }

**`meta`** (*required*) is the response metadata block described above.

**`name`** (*required*) is the name of the merchant.

**`type`** (*optional*) Type of merchants to search for. Examples include "restaurant," "cafe," "heath," "beauty," "department store," "groceries," etc.  

**`user_id`** (*required*) is the EVG login ID of the merchant. 

**`image`** (*optional*) is a URL to an image file for the suggested
  user.

**`lat`** (*required*) The merchant's latitude

**`lon`** (*required*) The merchant's longitude

**`distance`** (*required*) distance in meters from the passed coordinates. 

**`positive_rating`** (*optional*) number from 1-100 representing the trustworthiness of a merchant.

**`negative_rating`** (*optional*) number from 1-100 representing the lack of trustworthiness of a merchant.


## POST /make\_terminal_payment

#### example request

    POST {{url_base}}/make_terminal_payment
    {
	  "meta" : {...},
	  "amount": "45",
	  "description": "one cappicino + two lunches",
	  "payor": "facebook:123412124",
	  "PIN": "12346"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`amount`** (*required*) The amount of the transaction, in EVG credits, as a string.  Should be processed as a decimal value, but for now, only integral values of credits are allowed. **Must be positive**

**`description`** (*required*) description of the object required for payment

**`payor`** (*required*) The EVG user id of the person making the payment. The payee is the logged in user (i.e. the merchant).  For cards, payor is `qr:<QR code string>` (see section on User Ids)

**`PIN`** (*required*) The PIN of the person making the payment. 

** item_ids ** (*optional*) An array of items ids as strings (as provided by /items)

#### response

     {
	 "meta": {...},
	 "status": "insufficient_funds",
	 "status_message": "Your account balance is only 200ε, you cannot make this payment."
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`status`** (*required*) The result of the payment request.  Valid responses are:

- `success`: The payment was successful.

- `insufficient_funds`: The Wallet balance is insufficient to make the transaction.

- `invalid_user`: The username or card was not recognized.

- `error`: Some other error

**`status_message`** (*optional*) A message to display to the user modally to explain the result of the transaction




## POST /qr_status

QR Code actions will most likely happen in the context of a card that is issued by us and distributed by registered merchants, who will give away and then "load" the card by making a payment. We anticipate adding other workflows in the future.

#### example request

    POST {{url_base}}/qr_status
    
    {
	  "meta" : {...},
	  "qr_code" : "7xt6l1p23buNo1CR"
    }
    
##### Where:

**`meta`** (*required*) is the request metadata block described above.

**`qr_code`** (*required*)  The decoded QR code without any interpretation. 

#### response

     {
	 "meta": {...},
	 "status" : "valid",
	 "balance" : "138",
	 "image" : "https://images.evr.gr/38dadfjksiei.jpg",
	 "name" : "Joel Dietz"
     }

#### Where:

**`meta`** (*required*) is the response metadata block described above.

**`status`** (*required*) Valid responses are:

- `valid` The QR code is valid and can be charged.

- `not_registered` The QR code is not yet registered, but is eligible for registration.

- `invalid` The QR code is invalid and cannot be registered.

**`balance`** (*optional*) The balance of the card -- a positive integer as a quoted string.  Required if `status` is `valid`. 

**`image`** (*optional*) The URL to a photo of the user, if available. 

**`name`** (*optional*)  The natural name of the user, required if `status` is `valid`





# Products

## Common response

For all calls, the server replies with:

     {
	 "meta": {...},
          "items": [{"id": "42", "description": "description", "amount": "4"},
                        {"id": "13", "description": "unlucky number", "amount": "3"}]
     }

## Listing products

Current user:

    POST {{url_base}}/items
    
    {
	  "meta" : {...}
    }

For another user (read-only):

    POST {{url_base}}/items
    
    {
	  "meta" : {...},
          "payee": "facebook:..."
    }

## Creating, Updating, Deleting:

    POST {{url_base}}/items
    
    {
	  "meta" : {...},
          "items": [{"id": "42", "description": "updated description", "amount": "4"},
                        {"description": "new product (no id)", "amount": "4"},
                        {"id": "13", "deleted": "true"}]
    }

When updating, only created/updated/deleted items have to be sent. Untouched items will be preserved.


# Exchange Rates

Rates from Evergreen to other currencies

## POST /exchange_rates

Return a list of current exchange rates 


