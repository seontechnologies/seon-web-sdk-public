# Overview
You can integrate our device fingerprinting module directly into a website application, by using our JavaScript agent. Please use our CDN hosted script to ensure you always load the latest available version.

## Integration
1. Include the JavaScript Agent for example inside the <head> tags of your website or web app. You can also lazy-load it or execute it upon specific actions (e.g. clicking on Login, Payment, and Registration buttons, before calling the API). In this case, you must ensure that the module has been loaded successfully before invoking its methods.
2. Call the `seon.init()`
function on page load to get more data points for bot detection, behavioral analysis and more accurate intelligence signals.
3. Call the `seon.getSession(config)` function to generate the device  intelligence. After collecting all the available information, the function returns an encrypted base64 encoded payload.
4. Send the returned session payload string to your backend and add to the `session` property in your Fraud API request. The Fraud API call should be still executed if the `session` is missing, due to a non-executed JS snippet. Tip: Add timeout to JS and utilize Fraud API call after.

All the device intelligence signals will be available in the Fraud API response, and accessible on the Admin Panel of the Transactions Details page.

#### Example Integration
```html
<html>
  <head>
    ...
    <script src="[source_url]"></script>
  </head>
  <body>
    ...
  </body>
</html>
```

You can use the following script source URLs (`[source_url]`):
+ https://cdn.seondf.com/js/v6/agent.umd.js 
+ https://cdn.deviceinf.com/js/v6/agent.umd.js 
+ https://cdn.seonintelligence.com/js/v6/agent.umd.js

> [!Note]
> Without using the `init()` on page load you will still receive valid device intelligence signals with most of the functions but it will not contain the behavioral signals. Additionally, the bot detection and browser hash may be less precise.


> [!TIP]
> Fraud API documentation can be found [here](https://docs.seon.io/api-reference/fraud-api).

#### NPM integration
Alternatively you integrate the SDK through [NPM](https://www.npmjs.com/package/@seontechnologies/seon-javascript-sdk)

> [!WARNING]
> With this method you will have to keep the package updated yourself to include our latest features and bugfixes.

```sh
npm install @seontechnologies/seon-javascript-sdk
# or
yarn add @seontechnologies/seon-javascript-sdk
```

## Configuration

To configure the JavaScript module, you need to create a config object and call the seon.getSession(config) function. None of the parameters are required.


| Parameter  | Subparameter if an object | Description | Default value | Note |
| ------------- | ------------- | ------------- | ------------- | ----------------------------- |
|behavioralDataCollection||Settings for the behavioral biometrics data collection||*Details below*|
||targets|QuerySelector string that selects the targets for which the behavior biometrics should be enabled|`undefined`|If left undefined, it will track behavior on the whole page. To disable this feature, specify an empty string|
||formFilloutDurationTargetId|Selects the form by its element ID to measure the fillout time. Only the first matching element is considered|`undefined`|If left undefined, this datapoint won't be available|
| dnsResolverDomain || Other potential values: `seondfresolver.com`, `getdeviceinfresolver.com`, `seonintelligenceresolver.com` | *Can potentially change with minor versions!* Current default: `seondnsresolve.com` | Only set explicitly if potential changes are undesirable for you, please note that if your site uses CSP headers, then you must set a `connect-src` directive to allow requests to this domain and all subdomains  |
| fieldTimeoutMs  || Global timeout in milliseconds |`5000`| Rely on this option, rather than wrapping the 'getSession()' call in a timeout, because this way a partial result is still generated. Recommended minimum is 2000|
| geolocation  | | Geolocation configuration object | *Won't be collected by default* |  *Details below* |
||enabled|Shows whether geolocation is enabled or not.|`false`||
||highAccuracy| Enables high accuracy for the Geolocation API. It might slightly increase the fingerprinting time.|`true`|This affects how the built in browser API is used|
||canPrompt|Controls whether the SDK can generate a geolocation permission prompt in the browser|`false`|This is required for proper functionality in Safari|
||maxAgeSeconds|This option controls the maximum age in seconds of a cached position that is acceptable to return.|`0`|If set to 0, it means that the device cannot use a cached position and must attempt to retrieve the real current position|
||timeoutMs|Timeout for the Geolocation API to return the position of the device|`1000`|This timeout only works if 'canPrompt' is enabled. Also this option can be set to `Infinity`|
|networkTimeoutMs||Timeout for the network call in milliseconds|`2000`|For the most accurate results we utilize network requests to our services. This sets the maximum time that the SDK will wait for the response of these|
|referrer||Configures the `referrer` field in the resposne||*Details below*|
||maxLength|Maximum length of the URL|`128`|Allowed range: 0-1024|
||searchParams|Whether to include search parameters of the URL|`false`||
|region||Closest supported region of your user base|`eu`|Currently only Europe is supported|
|silentMode||Stops the SDK to trigger warnings & errors on the DevTools console.|`true`|Turning this off will allow the SDK to enable additional features.|
|throwOn||List of possible causes for the SDK to throw an error|`options`|By default the SDK only throws an error for an invalid 'options' object, but otherwise always runs to completion. Possible value: `geolocationPermissionDenied`|
|windowLocation||Configures the `windowLocation` field in the resposne||*Details below*|
||maxLength|Maximum length of the URL|`128`| Allowed range: 0-1024|
||searchParams|Whether to include search parameters of the URL|`false`||

> [!IMPORTANT]
> The `fieldTimeoutMs` is the global timeout for the fingerprinting, however this is not a hard limit, setting this value to a very low value (e.g. 100 ms) will probably still result in response times greater than the defined value because some operations must always complete for a valid result. 

#### Synergy between fieldTimeoutMs and other timeout parameters
+ Setting 'geolocation.canPrompt' to true and 'geolocation.timeout' will always force the SDK to wait for the user to either grant or deny the permission prompt (if given). Thus in this case the fingerprinting runtime will depend on the geolocation collection except when it finishes faster than the defined 'fieldTimeoutMs'
+ If the 'networkTimeoutMs' is greater than this option, and the network request is still pending when the defined timeout occurs, then the 'networkTimeoutMs' will be honored and the network request will still be awaited (until 'networkTimeoutMs').

#### Example configuration
```ts
// On page load:
seon.init();

const config = {
  geolocation: {
   canPrompt: false,
  },
  networkTimeoutMs: 2000,
  fieldTimeoutMs: 2000,
  region: 'eu',
  silentMode: true,
};
```

#### Example Usage
```ts
const session = await seon.getSession(config);
// 'session' variable holds the encrypted device fingerprint that should be sent to SEON
```

##  Behavioral features

Calling the `init()` method will enable behavioral analysis. The user behavior collection is started on the `init()` call and ends when the `getSession()` is called (behavioral data will be automatically included in the generated session string). Thus the recommended integration pattern is calling `init()` on the form load, and calling `getSession()` on form submit to analyze user behavior during a form fillout. 
Suspicious behavior is flagged in the `suspicious_flags` response field, which can contain the following values:
+ suspicious_keypress_characteristics
+ suspicious_mouse_movement
+ suspicious_form_fillout
+ paste_used
+ autofill_used

By default, user interaction is analyzed on the whole page. If you want to target specific input fields or forms for behavior analysis, you can customize it using the `behavioralDataCollection` `init()` configuration option:
```ts
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

The targeted elements MUST exist at the time of the `init()` call. Elements that match the selector, but added to the DOM after the `init()` call will NOT be part of the evaluation.

To disable behavioral data collection by the SDK altogether, you must specify an empty string for the `targets` option:

```ts
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
+ If you use CSP (Content Security Policy) headers on your site, you must allow the following domains in `connect-src` directive for full functionality based on your host configuration.
Default: `*.seondnsresolve.com` 
  + Alternatives:
    + seondf.com: `*.seondfresolver.com`
    + deviceinf.com: `*.deviceinfresolver.com`
    + seonintelligence.com: `*.seonintelligence.com`


# Changelog
## 6.1.0
+ Improved Multilogin detection, now detects latest X variant.
+ Implemented touch interaction based behavioral flags: `suspicious_touch_movement`, `suspicious_touch_speed`.
+ Bug fixes.

## 6.0.0
### Important Integration changes

+ #### Starting from v6 there is a change in [SEON’s API Policy](https://docs.seon.io/api-reference/api-policy). From now on SEON might introduce new fields in the SDK with minor versions. We advise you to integrate in a way that addition of new fields is handled gracefully.

+ #### The `brower_hash` field is calculated differently, resulting in different values for a given device. This means these values are going to break between versions.

+ #### `seon.getBase64Session()` has been renamed to `seon.getSession()`

+ #### For additional details on the changes of the integration and a comparison to the v5 open the [Migration guide](https://docs.seon.io/api-reference/migration-guides#js-sdk-v5-to-v6-guide)

### New features and improvements
+ Behavioral Biometrics features
+ Suspicious Flags
+ New datapoints and more robust fingerprinting
+ Smaller payload size
+ Improved screen and window resolution fields
+ API spoofing detection

### Response field changes
+ Numerous new fields were added, and some were changed, details in the migration guide. 

### Other
+ Internal changes to prepare for upcoming features and improvements like Remote Control Detection