---
title: Connector Scripting
sidebar: cyclr_sidebar
permalink: connector-scripting
keywords: action_data
redirect_from: "/custom-connector-scripting/"
tags: [connector-creation]
---

## Scripting

Cyclr supports Javascript as its scripting language, allowing you to manipulate data before it's sent as well as after it's been retrieved.  This can be useful when moving data between applications as what's valid in one, may not be valid in another.  Also, sometimes data just doesn't quite "fit".

Script can be used when building a Connector and on Steps in a Template or Cycle.

If you're working on a Cycle in the Builder and need to perform a change to some data, click the Step Setup button on a Step then either use Inline Script in a Mapping for something simple, or expand the Advanced Settings area and enter some Script to tie in to Cyclr's [Events](https://docs.cyclr.com/custom-connector-scripting#events) as described below.

For Inline Script, you must prefix the Javascript code with "=" (an equals sign), e.g.:
```javascript
=(100 * 2)
```
or
```javascript
=`[Mergefield]` === '' ? 'no value' : `[Mergefield]`;
```

It's best to use ` characters (backticks) around string values being merged in as that will prevent carriage returns and any quote characters from breaking your Script.

*Note: [Mergefield] above represents fields inserted by Cyclr in **Step Setup** when choosing **Type a Value** and selecting from the dropdowns.*


### Event Handlers

Events are triggered at certain points when a Cycle runs, allowing you to modify data using "event handlers" which are simply Javascript functions.

To add an event handler, put a Javascript function at the Connector or Method level, or in the Advanced Settings area of a Step in the Cycle Builder, using the event name as the name of the function, e.g.:

```javascript
function before_action() {
    /* Handle event here */
    return true;
}
```

Event handlers entered at the Connector level will be called for all of its Methods.  Event handlers entered at the Method level will only be called for that Method.

If you need to pass a value from a **before_action** handler to an **after_action** handler and you're not able to put it in the **method_request** object as part of the Request (the API being called might object to it), you can use the **method_request_mergefields** object as it's persisted across those two events. The **script_parameters** object, for example, is not persisted across any events.

### Event Handler Order

If an event handler exists at more than one level for the same event, i.e. Connector level as well as Method level, all of those event handlers are called.

This is useful if common processing of the data is required across all Methods using a Connector level **after_action** handler, but some Methods need further processing so an additional Method level **after_action** handler can also be used.

<br />

The order that Cyclr calls handlers for the same event is as follows:

Events beginning with "**before**" (such as before_action):

	Method -> Connector -> Builder Step

All other events (such as after_action):

	Connector -> Method -> Builder Step


### Global Objects

*   **method_response_fields**: Array containing a Method's Response Fields.


### Events

#### before_webhook

Called when a webook request has been received and before anything else is done. Method is used to decide if the request should be continued or return a custom message to the caller.

###### Global objects

*   **method_endpoint**: The webhook request URL
*   **method_request_headers**: The webhook request headers
*   **method_request**: The webhook request body
*   **method_request_parameters**: The webhook request parameters
*   **method_response_headers**: The response headers for the request
*   **method_response**: The response body for the request
*   **return**: true for the webhook to continue normal execution, false to stop execution of the request and send the response body/headers to the caller

#### after_webhook

Called immediately after a Request to a Webook has been received, whether the Cycle is currently running or stopped.

###### Global object

*   **method_response**: object that was POSTed to the Cyclr webhook
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **cycle_step_id**: ID of the step that is executing the script.
*   **cycle_id**: The ID of the cycle the script is running in
*   **cyclr_account_id**: The internal ID of the account the script is running in
*   **external_account_id**: The external ID of the account the script is running in
*   **return**: true for the webhook to continue normal execution, false to ignore the webhook request

#### before_action

Called before Cyclr makes a request to an external API.

If a Method uses Paging, this function is called before each page is retrieved.

###### Global objects

*   **method_request_headers**: HTTP headers for the request
*   **method_request_parameters**: Querystring parameters for the request
*   **method_request**: Object that will be posted to the third party API
*   **method_request_mergefields**: Mergefields for the request
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **cycle_step_id**: ID of the step that is executing the script.
*   **cycle_id**: The ID of the cycle the script is running in
*   **cyclr_account_id**: The internal ID of the account the script is running in
*   **external_account_id**: The external ID of the account the script is running in
*   **last_successful_run_date**: Time value indicating when a Step last ran without error.
*   **action_data**: An object used to persist data between some event handler functions, allowing data to be passed between them.  Accessible in before_action, after_action, after_action_paging, action_condition and after_error.
*   **return**: true to continue with the request to the third party API, false to abort the request (use throw for a more useful step error message)

#### after_action

Function is called when Cyclr has a response from an external API.

If a Method uses Paging, this function is called after each page is retrieved.

###### Global object

*   **method_endpoint**: The URL of the original request
*   **method_request**: object that was posted to the third party API
*   **method_request_mergefields**: mergefields for the request
*   **method_response_headers**: The response headers for the request
*   **method_response**: object that was received from the third party API.  If the Method uses paging, this contains only the current page's Response.
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **cycle_step_id**: ID of the step that is executing the script.
*   **cycle_id**: The ID of the cycle the script is running in
*   **cyclr_account_id**: The internal ID of the account the script is running in
*   **external_account_id**: The external ID of the account the script is running in
*   **action_data**: An object used to persist data between some event handler functions, allowing data to be passed between them.  Accessible in before_action, after_action, after_action_paging, action_condition and after_error.
*   **return**: true

#### after_action_paging

If this function is provided, it is called once after all pages of data have been retrieved, whether Paging has been implemented or not.

> The Paging referred to here is **Inbound**.  This function doesn't wait for **Outbound** paging to complete.

###### Global object

*   **method_request_headers**: The response headers for the request
*   **method_request_parameters**: parameters for the request
*   **method_request_mergefields**: mergefields for the request
*   **method_response**: object that contains all of the Response data.
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **cycle_step_id**: ID of the step that is executing the script.
*   **cycle_id**: The ID of the cycle the script is running in
*   **cyclr_account_id**: The internal ID of the account the script is running in
*   **external_account_id**: The external ID of the account the script is running in
*   **next_last_successful_run_date**: Assign to this variable to set the *last_successful_run_date* value a Step will use on its next run.
*   **action_data**: An object used to persist data between some event handler functions, allowing data to be passed between them.  Accessible in before_action, after_action, after_action_paging, action_condition and after_error.
*   **return**: true

#### after_error

Function is called when Cyclr received an error from an external API.

###### Global object

*   **method_error**: Details of the error, see: **Handle Errors from Third Party APIs** further down for more information on handling errors
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **cycle_step_id**: ID of the step that is executing the script.
*   **cycle_id**: The ID of the cycle the script is running in
*   **cyclr_account_id**: The internal ID of the account the script is running in
*   **external_account_id**: The external ID of the account the script is running in
*   **action_data**: An object used to persist data between some event handler functions, allowing data to be passed between them.  Accessible in before_action, after_action, after_action_paging, action_condition and after_error.
*   **return**: true

#### action_condition

Function is used to essentially combine a Method with a Decision Step, allowing a test to be performed that directs a Transaction down either the True or False exit points.  If this function is included in a method, Cyclr will add True and False exit points.

###### Global object

*   **method_response**: object that was received from the third party API.
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **cycle_step_id**: ID of the step that is executing the script.
*   **cycle_id**: The ID of the cycle the script is running in
*   **cyclr_account_id**: The internal ID of the account the script is running in
*   **external_account_id**: The external ID of the account the script is running in
*   **action_data**: An object used to persist data between some event handler functions, allowing data to be passed between them.  Accessible in before_action, after_action, after_action_paging, action_condition and after_error.
*   **return**: true for the Transaction to exit on the "True Route", false to exit on the "False Route"

#### before_oauth2_authorise

Function is called before Cyclr makes an OAuth 2 authorise request.

###### Global object

*   **method_endpoint**: URL for the OAuth authorise endpoint
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **return**: true

#### before_oauth2_token

Called before Cyclr makes an OAuth 2 access token request.

###### Global object

*   **method_request_headers**: HTTP headers for the request
*   **method_request**: Object that is going to be sent to the OAuth 2 access token endpoint
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **return**: true

#### after_oauth2_token

Called after Cyclr makes an OAuth 2 access token request.

###### Global object

*   **method_response**: response object that was received from the OAuth 2 access token request
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **return**: true

#### before_oauth2_refresh

Called before Cyclr makes an OAuth 2 refresh token request.

###### Global object

*   **method_request_headers**: HTTP headers for the request
*   **method_request**: request object that is going to be sent to the OAuth 2 refresh token request
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **return**: true

#### after_oauth2_refresh

Called after Cyclr makes an OAuth 2 refresh token request.

###### Global object

*   **method_response**: response object that was received from the OAuth 2 refresh token request.
*   **cycle_variables**: Allows access to Cycle variables.  Changes are not persisted.
*   **return**: true

### Functions

#### http_request

Function to make external HTTP requests.

When calling the `http_request` function, you provide a JSON object with the following properties:

*   method: HTTP method, e.g. GET, POST, DELETE, PUT
*   url: URL for the HTTP request
*   parameters: Querystring parameters
*   headers: HTTP headers
*   data: HTTP request data.  If sending JSON, you should use JSON.stringify() to serialize it.

Example:

```javascript
function after_action() {
	var response = http_request(
		{
			'method': 'POST',
			'url': 'https://someapi.com/createsomething',
			'headers':
			{
				'Authorization': 'Bearer ' + method_auth_value,
				'Content-Type': 'application/json',
				'Accept': 'application/json'
			},
			'data': JSON.stringify( { "MyData": "some value" } )
		}
	);

	return true;
}
```

The Response from an `http_request` call is returned as a JSON object with these properties:

*  status_code: the HTTP Status code returned
*  headers: any HTTP headers
*  content: the Response body
*  request: details of the Request that was made


#### btoa

Function to encode a string using Base64.

#### atob

Function to decode a Base64 encoded string.

#### cyclr_sign

Function to sign a string.

For example,
```javascript
var algorithm = 'HMAC-SHA1';
var signingKey = 'This is the signing key.';
var valueToSign = 'This is the string to sign.';

var signature = cyclr_sign(algorithm, signingKey, valueToSign);
```

Supported algorithms are: `HMAC-SHA1`, `RSA-SHA1`, `RSA-SHA224`, `RSA-SHA256`, `RSA-SHA384`, `RSA-SHA512`.

#### cyclr_csv_parse

Function to parse a CSV string.

```javascript
var csv = '1,2,3\na,b,c';
var delimiter = ',';
var hasHeader = false;

var csvRecords =  cyclr_csv_parse(csv, delimiter, hasHeader);;
```

#### Storage Functions

Cyclr provides several storage functions available for use in Script that can be used when developing a Connector, or on a Step in a Template or Cycle.

Data they work against is locked to the Connector they're called on.  i.e. if you write data using `cyclr_storage_set()` on a Step from one Connector, you cannot access that same data using `cyclr_storage_get()` on a Step from a different Connector.

The functions come in 2 flavours:

`cyclr_storage_...()` and `cycle_storage_...()`

The `cyclr_storage_...()` functions access the same data on any Steps in any Cycles for a particular Connector.

The `cycle_storage_...()` functions work in the same way, but their data is further restricted to the context of a particular Cycle.  If you write data in one Cycle, you cannot access it in another;

The storage functions available in their `cyclr_` versions are shown below.

Change `cyclr_` to `cycle_` to use their Cycle-restricted versions.

* cyclr_storage_list_values()
* cyclr_storage_delete_all()
* cyclr_storage_delete(key)
* cyclr_storage_get(key)
* cyclr_storage_set(key, value)
* cyclr_storage_append(key, value)
* cyclr_storage_list_keys()


### Exceptions

#### AuthRefreshException

Exception to force the OAuth 2 authentication token to be refreshed.

This is useful when the OAuth 2 endpoint doesn't return a definite token expiry time.

Upon getting this exception, Cyclr will call the OAuth 2 *Access Token URL* to get a new access token.

For example, an API returns 200 with an error code in the response when the token becomes invalid:

```javascript
function after_action() {
    if (typeof method_response.error !== 'undefined' &&
        method_response.error === 'invalid_grant') {
        throw new AuthRefreshException();
    }
    return true;
}
```

If an API returns a non-2xx HTTP status code when the auth token becomes invalid, you should throw *AuthRefreshException* in *after_error*:
```javascript
function after_error() {
    if (method_error.statusCode.toString() == 403) {
        throw new AuthRefreshException();
    }
    return true;
}
```

#### AuthSessionException

Exception to force the authentication session to be refreshed.

Upon getting this exception, Cyclr will call the *Post Install Property Value Lookup Method* to start a new session.

For example, an API returns 200 with an error code in the response when the session expires:

```javascript
function after_action() {
    if (typeof method_response.error_code !== 'undefined' &&
        method_response.error_code === 'You are not logged on.') {
        throw new AuthSessionException();
    }
    return true;
}
```

If an API returns a non-2xx HTTP status code when the auth session expires, you should throw *AuthSessionException* in *after_error*:
```javascript
function after_error() {
    if (method_error.statusCode.toString() == 403) {
        throw new AuthSessionException();
    }
    return true;
}
```

### Libraries

The following libraries are available within Cyclr's script engine:

> NB. It is not necessary to load these with a `require` call, they are ready to use in script.

#### Moment.js

Library Name: moment

Description: Parse, validate, manipulate, and display dates and times in JavaScript.
Cyclr also supports the Moment Timezone extension, which enables formatting and converting of dates in any timezone.

External Documentation: <https://momentjs.com/>, <https://momentjs.com/timezone/>


#### CryptoJS

Library Name: crypto-js

Description: JavaScript library of crypto standards.

Warning: The output of encrypted data is always in a hex string. Formatting options `CryptoJS.enc` are not supported when calling `toString`.

External Documentation: <https://github.com/brix/crypto-js>

### Connector scripting examples

#### Make External Requests

You can write a script to call external API endpoints. This is especially useful if the API returns a URL which contains the real response object.

For example, a webhook returns the following object to Cyclr:

```json
{
    "event": "object.updated",
    "api_url": "http://httpbin.org/get"
}
```

Use `http_request` to call `api_url` and replace the webhook response with the updated object:

```javascript
function after_webhook() {
    var request = {
        'method': 'GET',
        'url': method_response.api_url,
        'headers': {
            'Accept': 'application/json'
        }
    };

    var content = http_request(request).content;
    method_response = content;
    return true;
}
```
    

After calling `api_url`, Cyclr will then replace `method_response` with the content of the HTTP call.

Return `false` in the `after_webhook` function will stop Cyclr from running the webhook. You can use this trick to filter webhook events.

#### Transform Key Value Pairs

Making use of key value pair responses requires the use of scripting, consider an API that returns the below representation of a contact.

```json
{
    properties: [{
        "key": "email",
        "value": "example@example.com"
    }]
}
```

To access the email field we would add a field in the method response with a connector location of **properties.email**. However this would not work as the cyclr is looking in the response for a properties object with a property named email to get the value from. To solve the issue we would add the below function into the method scripts, this function will transform the properties array into an object with properties for each key value pair.

```javascript
function after_action() {
    var original = method_response.properties;
    method_response.properties = {};

    for (var i = 0; i < original.length; i++) {
        var item = original[i];
        if (item['key'] == void(0))
            continue;

        var val = item['value'];
        if (val == void(0))
            continue;

        method_response.properties[item['key']] = val;
    }

    return true;
}
```
    

Now when cyclr runs the method if will get the following result back and the **properties.email** field will work as expected.

```json
{
    properties: {
        "email": "example@example.com"
    }
}
```    

For a corresponding request method, e.g. adding a contact, we would need the below function in the method scripts to perform the data transformation in reverse.

```javascript
function before_action() {
    var original = method_request.properties;
    method_request.properties = [];

    for (var p in original) {
        method_request.properties.push({
            'key': p,
            'value': original[p]
        });
    }
    return true;
}
```
    
#### Modify Parameters

Besides the HTTP request body, you can also use scripting to modify HTTP headers (`method_request_headers`) and query string parameters (`method_request_parameters`).

```javascript
function before_action() {
    var xmlData = '<Records><Record>';

    for (var p in method_request) {
        xmlData += '<Field val=""' + p + '"">' + method_request[p] + '</Field>';
    }

    xmlData += '</Record></Records>';
    method_request_parameters.xmlData = xmlData;
    return true;
}
```    

In this example, we transformed the method request body to a XML string and saved the string as a new parameter called `xmlData`.

#### Handle Errors from Third Party APIs

The scripting engine can be used to catch and handle errors returned from third party APIs.

Cyclr exposes a received error response in the `after_error` function through the `method_error` object, which has these properties:

*   **statusCode** – the HTTP status code returned by the third party API
*   **reasonPhrase** – the reason phrase returned by the third party API
*   **content** – the body content of the response from the third party API
*   **isError** – indicates that the error is an error. default: true, set to false if using isWarning or isSuccess
*   **isWarning** – set to true for Cyclr to log the error as a warning
*   **isSuccess** – set to true to change the error to success, update content to contain the success step data

Example: change an error to a warning

```javascript
function after_error() {
    if (method_error.statusCode.toString() == 400 && method_error.reasonPhrase == 'Email Address not valid') {
        method_error.isError = false;
        method_error.isWarning = true;
    }
    return true;
}
```

Example: change an error to a success

```javascript
function after_error() {
    if (method_error.statusCode.toString() == 400 && method_error.reasonPhrase == 'Email Address not valid') {
        method_error.isError = false;
        method_error.isSuccess = true;
        method_error.content = '{}';
    }
    return true;
}
```    

### Limitations

*   Execution time: 60 seconds. Script running will time out after 60 seconds.
*   External HTTP requests: for security reasons, we will use the same authentication method as the connector and the same authentication value when the connector was installed by the user. You cannot use the script to access or modify the authentication value.
*   The **cycle_variables** object is only available through a Step's Advanced Settings area, and not through Inline Script.  Also, any changes made to it and its properties are not persisted.
