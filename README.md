# Overview
You can integrate our device fingerprinting module directly into a website application, by using our JavaScript agent. Please use our CDN hosted script to ensure you always load the latest available version.

## Integration
1. Include the JavaScript Agent for example inside the <head> tags of your website or web app. You can also lazy-load it or execute it upon specific actions (e.g. clicking on Login, Payment, and Registration buttons, before calling the API). In this case, you must ensure that the module has been loaded successfully before invoking its methods.
2. Call the `seon.init()`
function on page load to get more data points for bot detection, behavioral analysis and more accurate intelligence signals.
3. Call the `seon.getSession(config)` function to generate the device  intelligence. After collecting all the available information, the function returns an encrypted base64 encoded payload.
4. Send the returned session payload string to your backend and add to the `session` property in your Fraud API request. The Fraud API call should be still executed if the `session` is missing, due to a non-executed JS snippet. Tip: Add timeout to JS and utilize Fraud API call after.

All the device intelligence signals will be available in the Fraud API response, and accessible on the Admin Panel of the Transactions Details page.

> [!Note]
> Without using the seon.init() on page load you will still receive valid device intelligence signals with most of the functions but it will not contain the behavioral signals. Additionally, the bot detection and browser hash may be less precise.


> [!TIP]
> Fraud API documentation can be found [here](https://docs.seon.io/api-reference/fraud-api).

## Configuration

To configure the JavaScript module, you need to create a config object and call the seon.getSession(config) function. None of the parameters are required.


| Parameter  | Description | Default value | Note |
| ------------- | ------------- | ------------- | ------------- |
| fieldTimeoutMs  | Global timeout in milliseconds || Rely on this option, rather than wrapping the 'getSession' call in a timeout, because this way a partial result is still generated. |
| Content Cell  | Content Cell  | | |

#### Example configuration
```
```

##  Behavioral features

Calling the `seon.init()` method will enable behavioral analysis. The user behavior collection is started on the `seon.init()` call and ends when `seon.getSession()` is called (behavioral data will be automatically included in the generated session string). Thus the recommended integration pattern is calling `init` on the form load, and calling `getSession` on form submit to analyze user behavior during a form fillout. 
Suspicious behavior is flagged in the `suspicious_flags` response field, which can contain the following values:
+ suspicious_keypress_characteristics
+ suspicious_mouse_movement
+ suspicious_form_fillout
+ paste_used
+ autofill_used

By default, user interaction is analyzed on the whole page. If you want to target specific input fields or forms for behavior analysis, you can customize it using the `behavioralDataCollection` init configuration option:
```
// On load
seon.init({
  behavioralDataCollection: {
    targets: 'input[type="text"], .behavior', // querySelector string
    formFilloutDurationTargetId: "myForm", // select form with id 'myForm'
  }
});

// On form submit
await seon.getSession();
```

The targeted elements MUST exist at the time of the `init` call. Elements that match the selector, but added to the DOM after the `init` call will NOT be part of the evaluation.

To disable behavioral data collection by the SDK altogether, you must specify an empty string for the `targets` option:

```
// Disabling behavioral analysis
seon.init({
  behavioralDataCollection: {
    targets: '', // pass an emtpy string for targets
  }
});
```

## Payload
SEON JavaScript library collects device information and prepares an encrypted payload to use in Fraud API. If the information on client side is not readable, we’ll reveal in the Fraud API response and on the Admin Panel. Some fields can be `null`, if the actual browser does not support or return data for that specific data point. In every other case, data types are preserved.

## Common issues
+ The `session` is provided in the Fraud API request, but the `device_details` is null in the response and there is no device information on the Transaction details page. This means the encrypted payload is corrupted. Please look into your integration and check again.
+ If you use CSP (Content Security Policy) headers on your site, you must allow the following domains in connect-src directive for full functionality based on your host configuration.
Default: `*.seondnsresolve.com` 
  + Alternatives:
    + seondf.com: `*.seondfresolver.com`
    + deviceinf.com: `*.deviceinfresolver.com`
    + seonintelligence.com: `*.seonintelligence.com`


# Changelog
## 6.0.0
