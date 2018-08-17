# API

The back-end and API infrastructure is built upon the Loopback framework, a Node.js RESTful API framework with in-built API and database relation/model management.

All listed API requests can be viewed at `localhost:3000/explorer/` when `npm run dev:server` is running in the background. Additionally, all up-to-date public API requests are listed and have examples at http://178.62.80.203:3000/explorer/. This is a Loopback API explorer running off of the Perfocal beta server hosted on Digital Ocean (as all of Perfocal's servers are). All requests have example values and model definitions listed as well as a response shown in JSON format after an example request has been prossessed either successfully or an error is thrown.

Live and Beta production server instances are handled and contained by **Docker** in images named `perfocal.com/perfocal:x.y.z` where `x`, `y` and `z` are major, minor and patch version numbers respectively.

All major database models and server methods can be viewed in the `server/models/` folder. Please note that server methods that have empty export functions do not have custom methods and only use default Loopback REST methods.
All custom methods are explained in detail in the list below. Some methods listed below are private and therefore do not have request URLs, these methods are therefore only used to perform specific functions within some other, public, requests.

-----

## PerfocalUser

`PerfocalUser`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **firstName** - `string` - first_name - User's first name
- **lastName** - `string` - last_name - User's last name
- **email** - `string` - email - User's email
- **emailVerified** - `boolean` - email_verified - Status of email verification
- **verificationToken** - `string` - verification_token - Token used in email verification, `null` if email is verified
- **cityName** - `string` - city_name - Name of the city the user is based in, defaults to London
- **phone** - `string` - phone - User's phone number
- **stripeCustomer** - `string` - stripe_customer - ID of stripe customer used in customer payment
- **password** - `string` - password - User's hashed password string
- **username** - `string` - username - User's username, automatically generated on creation of user object
- **realm** - `string` - realm - type of user: '**customer**', '**photographer**' or '**operator**'
- **created** - `date` - created - Date-time of creation of user

### Methods

#### `User.resetPassword(email)`

**Request path: `POST` `/users/reset-password`**

**Accepts: `email: string` via `formData`**

Attempts to find a `PerfocalUser` where `PerfocalUser.email` == `email`, if a user is found, a `email:reset-password` job is created to send out a reset-password link, otherwise a `404: Not Found` error is thrown out detailing that the email does not exist.

**Returns `true`**

#### `User.confirm(uid, token, redirect)`

**Request path: `GET` `/users/confirm`**

**Accepts: `uid`: `string` via `query`, `token`: `string` via `query`, optional `redirect`: `string` via `query`**

Confirms user registration with an identity verification token

**No return value**

#### `User.prototype.cards()`

**Request path: `GET` `/users/{id}/cards`**

**Accepts: `id`: `string` via `path`**

Attempts to find and return all valid (CVC check passed) cards connected to the user's Stripe account.
If the user does not have a `stripeCustomer` attribute, an empty array is returned.

**Returns `array`**

---

## Photographer

`Photographer`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **userId** - `number` - user_id - Foreign key for user model, links photographer to user
- **firstName** - `string` - first_name - User's first name
- **lastName** - `string` - last_name - User's last name
- **email** - `string` - email - User's email
- **phone** - `string` - phone - User's phone number
- **phoneVerified** - `boolean` - phone_verified - Status of phone verification
- **verificationToken** - `string` - verification_token - Token used in phone verification, `null` if verified
- **gender** - `string` - gender - Photographer's gender: '**female**', '**male**', '**other**'
- **bio** - `string` - bio - Short photographer bio
- **birthday** - `date` - birthday - Photographer's birthdate
- **address** - `string` - address - Photographer's work address as a string
- **lat** - `number` - lat - Photographer's work address latitude coordinate
- **lng** - `number` - lng - Photographer's work address longitude coordinate
- **postcode** - `string` - postcode - Photographer's work address postcode, *please note that this is not used in the newly redesigned matching process*
- **professionalSince** - `string` - prof_since - Year the photographer has worked professionally
- **paidExperience** - `boolean` - paid_exp - If photographer has paid experience, *please note that this value is deprecated and is not used anywhere within the platform*
- **website** - `string` - website - Photographer's portfolio or personal website URL featuring their work
- **experienceWithPlatforms** - `string` - exp_platforms - *This value is deprecated and is not used anywhere within the platform*
- **cameras** - `array` - cameras - List of photographer's cameras
- **lenses** - `array` - lenses - List of photographer's lenses
- **accessories** - `array` - accessories - List of photographer's photography accessories
- **avatar** - `string` - avatar - Photographer's avatar URL
- **stripeId** - `string` - stripe_id - Photographer's stripe account ID
- **created** - `date` - created - Date-time of creation of user object

### Methods

#### `Photographer.prototype.updatePhone(phone)`

**Request path: `/photographers/{id}/phone`**

**Accepts: `id`: `string` via `path`, `phone`: `string` via `formData`**

Formats photographer's phone number and checks if the input phone number has been taken or already confirmed. If the checks have been passed, the photographer is then sent an SMS message to that number containing a verification token.

**Returns `Photographer`**

#### `Photographer.findByLocation(lat, lng, filter)` - Private

**Accepts: `lat`: `number`, `lng`: `number`, `filter`, `object`**

Returns a list of up to 50 photographers near a given location via `lat` and `lng` coordinates. These photographers are ranked according to their proximity to the location and their certificate score for a given category passed through `filter`. 

The function finds all photographer certificates that match a given `PhotoCategory` and include `Photographer`. If there are photographers, the list is then sorted by a function `sortPhotogs(photogList, lat, lng)`:

- Iterate through `photogList`:
  - Get the distance between the event lat/lng and photographer's lat/lng
  - If the distance is less than or equal to 100km,
    - The a rating/score is given to that photographer.
    - The photographer is then pushed to an unsorted list
- The unsorted list is then sorted by the rating/score of each photographer and then the photographer details are retrieved from `photogList`

**Returns `array`**

---

## Event

`Event`

### Properties

- **property** - `type` - column_name - description
- **id** - `string` - id - Event's unique identifier formed using 6 random alphanumerics
- **created** - `date` - created - Timestamp for creation of event
- **sceneId** - `number` - scene_id - Id of `Scene`/`Photoscene` the Event is using (reword?). Foreign key for `Photoscene`
- **customerId** - `number` - customer_id - Id of customer user that created the event. Foreign key for `User`
- **firstName** - `string` - first_name - Customer's first name
- **lastName** - `string` - last_name - Customer's last name
- **email** - `string` - email - Customer's email
- **phone** - `string` - phone - Customer's phone number
- **starts** - `date` - starts - Timestamp for the beginning of the event
- **address** - `string` - address - Event address as a string
- **lat** - `number` - lat - Event address latitude coordinate
- **lng** - `number` - lng - Event address longitude coordinate
- **postcode** - `string` - postcode - Event address postcode, *please note that this value is not used in the newly update matching process*
- **numberOfGuests** - `string` - numberofguests - Approximate number of guests at the shoot event
- **venueType** - `string` - venuetype - Type of venue where the event will take place, '**indoor**', '**outdoor**', '**both**', '**not sure**'
- **addons** - `array` - addons - List of addons that the customer has purchased with the event
- **additionalDetails** - `string` - additional_details - Any additional details a customer provides for photographer/admin
- **status** - `string` - status - Current event status: '**created**', '**paid**', '**matching**', '**stopping**', '**stopped**', '**assigned**', '**delivering**', '**delivered**', '**delivered**', '**fulfilled**', '**cancelled**', '**expired**'
- **packageId** - `number` - package_id - Id of `Package/Pricing` the Event is using (reword?). Foreign key for `Pricing`
- **duration** - `string` - duration - String for how long the Event lasts for, '**30 minutes**', '**1 hour**', '**2 hours**', '**3 hours**', '**4 hours**', '**8 hours**'
- **shareLink** - `string` - share_link - Randomly generated character string for public customer galleries
- **allowDownload** - `boolean` - allow_download - Status for allowing guests to download from public galleries

### Methods

#### `Event.prototype.generateRequestQueue()` - Private

**Accepts: `null`**

Attempts to generate a list of `EventRequest`. Calls `Photographer.findByLocation()` to get a list of potential photographers and creates an `EventRequest` for each photographer in the list.

**Returns `number`**

#### `Event.prototype.startMatchMaking()`

**Request path: `POST` `/events/{id}/queue`**

**Accepts: `id`: `string` via `path`**

Starts the match making server jobs, updates the event status to 'matching'. If there aren't any requests, an error is thrown in the server logs.

**Returns `number`**

#### `Event.prototype.payout(amount)`

**Request path: `POST` `/events/{id}/payout`**

**Accepts: `id`: `string` via `path`, optional `amount`: `number` via `formData`**

If `Event.status` == `'delivered'`, finds an `Order` where `eventId` == `Event.id`. If there is an `Order`, a Stripe transfer is attempted to the photographer's Stripe account via their Stripe id. `Payment` and `Order` have their attributes updated to reflect a successful payout. Two email jobs are also created to notify the customer and photographer that the event is now fulfilled and closed.

**Returns `Event`**

#### `Event.prototype.cancel`

**Request path: `POST` `/events/{id}/cancel`**

**Accepts: `id`: `string` via `path`**

If the event is refundable ('paid', 'stopped', 'matching' or 'assigned), `EventRequests` are made invalid ('closed'/'unsent') if the event is 'matching', `Order` is updated to 'cancelled' if the event is 'assigned'. Up to two email jobs are made to inform the customer, and photographer, about the cancellation of the event.

**Returns `Event`**

#### `Event.prototype.finishDelivery()` - Private

**Accepts: `null`**

Updates order and event statuses to after all photos/albums have been uploaded and delivered. Creates two email jobs to inform the customer and photographer about the delivery of photos. Generates a random `shareLink` string for the customer's public gallery.

**No return value**

#### `Event.prototype.createZIP(type)`

**Request path: `POST` `/events/{id}/zip`**

**Accepts: `id`: `string` via `path`, optional `type`: `string` via `formData`**

*If zip `type` is not included, it defaults to `'standard'`.*
Removes existing album of `type` and creates `worker:gallery-zip` job which gets all images of `type` and `eventId` and creates a zip folder which is then uploaded to AWS S3.

**Returns `true`**

#### `Event.prototype.generateShareLink()`

**Request path: `POST` `/events/{id}/generate-share-link`**

**Accepts: `id`: `string` via `path`**

Generates a random string of 10 alphanumerics and updates `Event.shareLink` attribute.

**Returns `string`**

#### `Event.prototype.createThumbnails(type)`

**Request path: `POST` `/events/{id}/thumbnails`**

**Accepts: `id`: `string` via `path`, optional `type`: `string` via `formData`**

*If image `type` is not included, it defaults to `'standard'`*
Creates `worker:create-thumbnails` job which gets all images of `type` and `eventId` and generates resized images depending on image `type`.

**Returns `true`**

#### `Event.prototype.freeCheckout(promocode)`

**Request path: `POST` `/events/{id}/free-checkout`**

**Accepts: `id`: `string` via `path`, optional `promocode`: `string` via `formData`**

Updates `Event.status` to `'paid'`. Creates email job to inform customer that the booking has been received and matching will start soon. If a promocode with limited uses is provided, the number of uses of that promocode is reduced by 1.

**No return value**

---

## EventRequest

`Event Request`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **created** - `date` - created - Timestamp of creation of event-request
- **eventId** - `string` - event_id - Id of event. Foreign key for `Event`
- **status** - `string` - status - Current event-request status: '**created**', '**pending**', '**overdue**', '**accepted**', '**rejected**', '**waitlist**', '**unsent**'
- **rank** - `number` - rank - Position of event-request in the list/queue
- **photographerId** - `number` - photographer_id - Id of photographer that is related to the event-request. Foreign key for `Photographer`
- **respondToken** - `string` - respond_token - Random string used to generate the event-request URL link
- **responded** - `date` - responded - Timestamp of photographer response
- **overdue** - `date` - overdue - Timestamp of end of exclusive window

### Methods

#### `EventRequest.prototype.respond(action, token)`

**Request path: `POST` `/event-requests/{id}/respond`**

**Accepts: `id`: `string` via `path`, `action`: `string` via `formData`, `token`: `string` via `formData`**

If `action` == `'accept'` and the `EventRequest` is within the exclusive window ('pending'), an `Order` is created and all other `EventRequests` for that event are set as 'closed'/'unsent'. If `EventRequest` is outside the exclusive window ('overdue'/'rejected'), the `EventRequest` is added to a waitlist. If `action` == `reject`, `EventRequest.status` is updated to 'rejected' and the match-making process moves onto the next `EventRequest` using `EventRequest.next()`

**Returns `EventRequest`**

#### `EventRequest.prototype.next()` - Private

**Accepts: `null`**

Gets the next `EventRequest` in the queue

**Returns `EventRequest`**

#### `EventRequest.prototype.prev()` - Private

**Accepts: `null`**

Gets the previous `EventRequest` in the queue

**Returns `EventRequest`**

---

## Order

`Order`

### Properties

- **property** - `type` - column_name - description
- id - `number` - id - Unique identifier
- created - `date` - created - Timestamp of creation of order
- **customerId** - `number` - customer_id - Id of the related event's customer.
- **photographerId** - `number` - photographer_id - Id of the photographer assigned to the event. Foreign key for `Photographer`
- **eventId** - `string` - event_id - Id of the event that contains the order. Foreign key for `Event`
- **fee** - `number` - fee - Photographer's payout fee
- **status** - `string` - status - Current order status: '**created**', '**uploaded**', '**overdue**', '**cancelled**'
- **overdue** - `date` - overdue - Timestamp for when the order is overdue
- **editorsNote** - `string` - editors_note - Note **for the editor** left by the photographer
- photographersNote - `string` - photogs_note - Note **for the photographer** left by the editor

---

## Certificate

`Certificate`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **created** - `date` - created - Timestamp of creation of certificate
- **photographerId** - `number` - photographer_id - Id of the photographer that has the certificate. Foreign key for `Photographer`
- **requestId** - `number` - req_id - Id of the certificate-request created by the photographer. Foreign key for `CertificateRequest`
- **categoryName** - `string` - category_name - Name of the photography category certificate that the photographer has applied for
- **score** - `number` - score - Judges how good the photographer is. This value is used in determining their rank in the event-request queue

---

## CertificateRequest

`CertificateRequest`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **submitted** - `date` - submitted - Timestamp of creation of certificate request
- **status** - `string` - status - Current certificate request status: '**pending**', '**accepted**', '**rejected**'
- **photographerId** - `number` - photographer_id - Id of photographer that has created a certificate request. Foreign key for `Photographer`
- **categoryName** - `string` - category_name - Name of the photography category certificate that the photographer has applied for. Foreign key for `PhotoCategory`
- **photos** - `array` - photos - List of photos that the photographer has submitted with their certificate request

### Methods

#### `CertReq.prototype.reject()`

**Request path: `POST` `/certificate-requests/{id}/reject`**

**Accepts: `id`: `string` via `path`**

Updates `CertificateRequest.status` to `'rejected'` and creates email job informing photographer about the rejected application

**Returns `object`**

#### `CertReq.prototype.accept()`

**Request path: `POST` `/certificate-requests/{id}/accept`**

**Accepts: `id`: `string` via `path`**

Updates `CertificateRequest.status` to `'accepted'` and creates email job informing photographer about the accepted application

**Returns `object`**

---

## PhotoScene

`PhotoScene`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **name** - `string` - name - Name of the scene
- **categoryName** - `string` - category_name - Name of the `PhotoCategory` that the scene belongs to. Foreign key for `PhotoCategory`
- **purpose** - `string` - purpose - What customer category the scene falls under: '**Personal**', '**Business**'
- **description** - `string` - description - Brief copy-text regarding what the scene is about and who it is for
- **image** - `string` - image - URL for the background image in the booking form

---

## Pricing

`Pricing`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **name** - `string` - name - Pricing package name
- **price** - `number` - price - Pricing package cost in pence (obviously)
- **currency** - `string` - currency - Currency to be used in the transaction, current default is 'gbp'. Stripe and `Payment` methods must be updated to check for currency property and charge accordingly
- **description** - `string` - description - Short copy-text outlining estimated photo number and duration
- **photographerFee** - `number` - photographer_fee - Photographer pay amount in pence
- **duration** - `string` - duration - Shoot time of the package
- **mostPopular** - `boolean` - mostpopular - Flag that manages if a package is the most popular in a particular scene
- **sceneId** - `number` - scene_id - Id of the scene that the package is related to. Foreign key for `Photoscene`

---

## Payment

`Payment`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **created** - `date` - created - Timestamp of creation of payment
- **amount** - `number` - amount - Total amount to charge, includes Photographer's fee, Platform fee and Stripe fee
- **fee** - `number` - fee - Photographer pay amount in pence
- **refund** - `number` - refund - Amount of money refunded back to customer's card
- **payout** - `number` - payout - Amount of money payed out to photographer
- **email** - `string` - email - Customer's email that was used to make the booking. This email is also used in the Stripe email receipt
- **released** - `date` - released - Timestamp of release of photographer payment
- **refunded** - `date` - refunded - Timestamp of refund back to customer
- **stripeToken** - `string` - stripe_token - Stripe token or card Id used to attempt to charge the customer's card
- **stripeCharge** - `string` - stripe_charge - Stripe charge Id after customer has successfully paid for the service
- **stripeTransfer** - `string` - stripe_transfer - Stripe transfer Id after photographer has been successfully payed out
- **status** - `string` - status - Current payment status: '**CREATED**', '**RELEASED**', '**REFUNDED**'
- **eventId** - `string` - event_id - Id of the event that the payment is for. Foreign key for `Event`
- **last4** - `string` - last_4 - Last four digits of the customer's card used to make the payment
- **promocode** - `string` - promocode - Valid promcode string used by the customer
-  **type** - `string` - type - Type of payment made: '**booking**', '**addon**'

---

## Promocode

`Promocode`

### Properties

- **property** - `type` - column_name - description
- **id** - `string` - id - Unique identifier. Used as the promocode name. This can not be updated via API `PATCH` calls
- **amount** - `number` - amount - Discount amount the promocode is worth. **Percentages go up to `100`, flat amounts are given in pence**
- **expires** - date - expires - Timestamp of expiry of promocode
- **discount** - `string` - discount - Type of discount the promocode gives: '**amount**', '**percent**'
- **uses** - `number` - uses - Number of uses that the promocode is valid for. `null` gives an unlimited number of uses
- **minSpend** - `number` - min_spend - Minimum spending amount that the promocode can be used with. `null` means the promocode is valid for all prices
- **validScenes** - `array` - valid_scenes - List of valid scenes that can be used with the promocode. `null` means the promocode can be used with any scene
- **validAddons** - `array` - valid_addons - List of valid addons that can be used with the promocode. `null` means the promocode can be used with any addon

---

## Image

`Image`

### Properties

- **property** - `type` - column_name - description
- **id** - `number` - id - Unique identifier
- **name** - `string` - name - Randomised name (UUID-v4) of the image
- **originalName** - `string` - original_name - Name the image is originally uploaded with
- **type** - `string` - type - Image size: '**original**', '**thumbnail**', '**web**', '**standard**', '**large**'
- **orderId** -`number` - order_id - Id of the order that the image is uploaded to. Foreign key for `Order`
- **eventId** - `string` - event_id - Id of the event that the image is uploaded to. Foreign key for `Event`
- **bucket** - `string` - bucket - Name of the AWS S3 bucket that the image is uploaded to
- **albumId** - `string` - album_id - Id/name of the album the image is part of

### Methods

#### `Image.destroyAll()`

**Request path: `DELETE` `/images/`**

**Accepts: `where`: `object` via `query`**

Deletes all images matching the `where` filter.

**No return value**

---

## Album

`Album`

### Properties

- **property** - `type` - column_name - description
- **id** - `string` - id - Unique identifier
- **created** - `date` - created - Timestamp of creation of album
- **type** - `string` - type - Type of images in album: '**original**', '**web**', '**standard**', '**large**'
- **bucket** - `string` - bucket - Name of the AWS S3 bucket that the album is uploaded to
- **eventId** - `string` - event_id - Id of the event that the album is part of

### Methods

#### `Album.prototype.download(res, expires)`

**Request path: `/albums/{id}/download`**

**Accepts: `id`: `string` via `path`, `res`: `object` via `res`, optional `expires`: `number` via `query`**

*If `expires` is not included, `expires` defaults to `3600`*
Gets signed download URL from AWS S3 bucket

**Returns `302 HTTP redirect`**

---

## PhotoCategory

`PhotoCategory`

### Properties

- **property** - `type` - column_name - description
- **name** - `string` - name - Photography category name
- **description** - `string` - description - Short copy-text outlining the category and what is required from the photographer
- **image** - `string` - image - URL for the background image in the application forms

---

# Jobs

All server jobs are listed under `server/jobs.js` and are handled by 'Kue', a Node.js job queue library. This allows the server to queue up several different jobs without having to wait for internal `async/await` or callbacks to move through a job queue.

These jobs allow the server to run tasks which are separate from the API service but perform critical tasks such as sending emails, match-making, image resizing and album zipping.

Listed below are the more important and, sometimes, more complicated server jobs.

### `match-making`

Gets `EventRequest` from given request `id`.
Checks the waitlist for `EventRequests`. If there is someone in the waitlist, first place gets 'awarded' the job and any others are sent `email:photog/job-taken`. 
Sets the previous `EventRequest.status` to `overdue`.
Sets an overdue time for current-time + 5 minutes unless the current time is after 11PM (23 hours), then the overdue time is set to 8:05AM of the next day.
Creates `send-event-request` with necessary request and photographer information.

### `send-event-request`

Creates a `match-making` job with a delay of roughly 5 minutes (difference between `EventRequest` overdue time and current time) for the next requestId in the list. Otherwise, creates a `last-check-waitlist` job with the same delay but using the given requestId, not the next one in line.
Creates `sms:photog/new-job` and `email:photog/new-job` jobs to send the photographer a link with event information and buttons to either accept or reject the request.

### `last-check-waitlist`

Checks the waitlist for `EventRequests`. If there is someone in the waitlist, first place gets 'awarded' the job and any others are sent `email:photog/job-taken`. Otherwise, all `EventRequests` for that event are set to `pending` and the first photographer to accept gets the job, regardless of their position in the list.

### `worker:gallery-zip`

Gets all images of `type` and `eventId`.
Loops through list of images, updates image name depending on the album and image `type` and appends the downloadStream of the image to the album bundle.
Once all images have been bundled, it is finalised via `bundle.finalize()` and uploaded to AWS S3 via a multipart upload of up to five 20MB concurrent part uploads.
When the album has been uploaded, an `Album` object is created and all necessary `Images` are updated to include the correct `albumId`.
Once all edited image albums have been zipped and uploaded, the job then calls `Event.finishDelivery()` to complete the job and remove itself from the queue.

### `worker:create-thumbnails`

Gets all `large` images and resizes each image depending on which image `type` is passed as a parameter; `thumbnail`, `web` or `standard`.
All existing images of that `type` are then deleted using `Image.destroyAll()`.
Each new image is then given a UUID filename and the `originalName` is updated to include the new image `type`. The image is then uploaded to AWS S3 and an `Image` object is created.
When all images have been resized and uploaded, the job calls `Event.createZIP()` to generate a zipped album for the image `type`
