This specification document is to establish the technical and compliance standards/protocols that are necessary for a biometric device to be used in MOSIP solutions.

API specification version:  **1.0**

Published Date: September 28, 2020

## Revision Note
Publish Date | Revision
-------------|--------
28-Sep-2020 | This is the first formal publication of this document as a version-ed specification. Earlier draft are superseded by this document.

## Glossary of Terms
* Device Provider - An entity that manufactures or imports the devices in their name. This entity should have legal rights to obtain an organization level digital certificate from the respective authority in the country.
* Foundational Trust Provider - An entity that manufactures the foundational trust module.
* Device - A hardware capable of capturing biometric information.
* SBI 2.0 Certified Device / SBI 2.0 Device - A device certified as capable of performing encryption in line with this spec in its trusted zone.
* SBI 1.0 Certified Device / SBI 1.0  Device - A device certified as one where the encryption is done on the host machine device driver or the SBI.
* Foundational Trust Provider Certificate - A digital certificate issued to the "Foundational Trust Provider". This certificate proves that the provider has successfully gone through the required Foundational Trust Provider evaluation. The entity is expected to keep this certificate in secure possession in an HSM. All the individual trust certificates are issued using this certificate as the root. This certificate would be issued  by the countries in conjunction with MOSIP.
* Device Provider Certificate - A digital certificate issued to the "Device Provider". This certificate proves that the provider has been certified for SBI 1.0/SBI 2.0 respective compliance. The entity is expected to keep this certificate in secure possession in an HSM. All the individual trust certificates are issued using this certificate as the root. This certificate is issued by the countries in conjunction with MOSIP.
* Registration - The process of applying for a Foundational Id.
* KYC - Know Your Customer. The process of providing consent to perform profile verification and update.
* Auth - The process of verifying one’s identity.
* FPS - Frames Per Second
* Management Server - A server run by the device provider to manage the life cycle of the biometric devices.
* Device Registration - The process of registering the device with MOSIP servers.
* Signature - All signature should be as per RFC 7515.
* header in signature - Header in signature means the attribute with "alg" set to RS256 and x5c set to base64encoded certificate.
* payload is the byte array of the actual data, always represented as base64urlencoded.
* signature - base64urlencoded signature bytes
* ISO Format Time - ISO 8601 with format yyyy-mm-dd HH:MM:ssZ
* MOSIP Devices - All devices that collect biometric data for MOSIP should operate within the specification of this document.

---

## Device Specification
The specification documentation provides compliance guidelines to devices for them to work with MOSIP. The compliance is based on device capability, trust and communication protocols. A MOSIP compliant device would follow the standards established in this document. It is expected that the devices are compliant to this specification and tested and validated. The details of each of these are outlined in the subsequent sections.

### Device Capability
The MOSIP compliant device is expected to perform the following:
* Should have the ability to collect one or more biometric
* Should have the ability to sign the captured biometric image or template.
* Should have the ability to protect secret keys
* Should have no mechanism to inject the biometric

For details about bometric data specifications please view the page [MOSIP Biometric Data Specification](Biometric-Data-Specification.md).

---

## Device Trust
MOSIP compliant devices provide a trust environment for the devices to be used in registration, KYC and AUTH scenarios. The trust level is established based on the device support for trusted execution.

* SBI 1.0 - The trust is provided at the software level. No hardware related trust exist. This type of compliance is used in controlled environments.
* SBI 2.0 - The trust is provided by a secure chip with secure execution environment.

### Foundational Trust Module (FTM)
The foundational trust module would be created using a secure microprocessor capable of performing all required biometric processing and secure storage of keys. The foundational device trust would satisfy the below requirements.

* The module has the ability to securely generate, store and process cryptographic keys.
* Generation of asymmetric keys and symmetric keys with TRNG.
* The module has the ability to protect keys from extraction.
* The module has to protect the keys from physical tampering, temperature, frequency and voltage related attacks.
* The module could withstand against Hardware cloning.
* The module could withstand probing attacks
* The module provides memory segregation for cryptographic operations and protection against buffer over flow attacks
* The module provides ability to withstand cryptographic side channel attacks like Differential Power analysis attacks, Timing attacks.
* CAVP validated implementation of the cryptographic algorithm.
* The module has the ability to perform a cryptographically validate-able secure boot.
* The module has the ability to run trusted applications.

The foundational device trust derived from this module is used to enable trust-based computing for biometric capture.  The foundational device trust module provides for a trusted execution environment based on the following:

* Secure Boot
    * Ability to cryptographically verify code before execution.
    * Ability to check for integrity violation of the module/device.
    * Halt upon failure.
    * Ability to securely upgrade and perform forward only upgrades, to thwart downgrade attacks.
    * SHA256 hash equivalent or above should be used for all hashing requirements
    * All root of trust is provisioned upon first boot or before.
    * All upgrades would be considered success only after the successful boot with proper hash and signature verification.
    * The boot should fail upon hash/signature failures and would never operate in a intermediary state.
    * Maximum of 10 failed attempts should lock the upgrade process and brick the device. However chip manufactures can decide to be less than 10.
* Secure application
    * Ability to run applications that are trusted.
    * Protect against downgrading of applications.
    * Isolated memory to support cryptographic operations.
    * All trust are anchored during the first boot and not modifiable.

#### Certification
The FTM should have a at least one of the following certifications in each category to meet the given requirement.

##### Category: Cryptographic Algorithm Implementation
* CAVP (RSA, AES, SHA256, TRNG (DRBGVS), ECC)

{% hint style="info" %}
The supported algorithm and curves are listed [here](#cryptography)
{% endhint %}

##### Category: FTM Chip
* FIPS 140-2 L3 or above
* PCI PTS 5 or above (Pre-certified)
* Common Criteria (EAL4 and above)

##### Category: Tamper
* For SBI 2.0 level compliance the FTM should support tamper evidence.

#### Threats to Protect
The FTM should protect against the following threats.

* Hardware cloning attacks - Ability to protect against attacks that could result in a duplicate with keys.
* Hardware Tamper attacks
    * Physical tamper - No way to physically tamper and obtain it secrets. 
    * Voltage & frequency related attacks - Should shield against voltage leaks and should prevent against low voltage. The FTM should always be in either of the state - "operational normally" or "inoperable". The FTM should never be operable when its input voltages are not met.
    * Temperature attacks on crypto block - Low or high the FTM is expected to operate or reach inoperable state and no state in between.
* Differential Power Analysis attack.
* Probing attacks - FTM should protect its surface area against any probe related attacks.
* Segregation of memory for execution of cryptographic operation (crypto block should be protected from buffer overflow type attacks).
* Vulnerability of the cryptographic algorithm implementation.
* Attacks against secure boot & secure upgrade.
* TEE/Secure processor OS attack.

#### Foundational Trust Module Identity
Upon an FTM provider approved by the MOSIP adopters, the FTM provider would submit a self signed public certificate to the adopter. Let us call this as the FTM root. The adopter would use this certificate to seed their device trust database. The FTM root and their key pairs should be generated and stored in FIPS 140-2 Level 3 or more compliant devices with no possible mechanism to extract the keys.  The foundational module upon its first boot is expected to generate a random asymmetric key pair and provide the public part of the key to obtain a valid certificate.  The FTM provider would validate to ensure that the chip is unique and would issue a certificate with the issuer set to FTM root.  The entire certificate issuance would be in a secured provisioning facility. Auditable upon notice by the adopters or its approved auditors.  The certificate issued to the module will have a defined validity period as per the MOSIP certificate policy document defined by the MOSIP adopters. This certificate and private key within the FTM chip is expected to be in it permanent memory.

{% hint style="info" %}
The validity for the chip certificate can not exceed 20 years from the date of manufacturing.
{% endhint %}

### Device
Mosip devices are most often used to collect biometrics. The devices are expected to follow the specification for all level of compliance and its usage. The mosip devices fall under the category of Trust Level 3 (TL3) as defined in MOSIP architecture. At TL3 device is expected to be whitelisted with a fully capable PKI and secure storage of keys at the hardware.

* SBI 1.0 - A device can obtain SBI 1.0 certification when it uses software level cryptographic library with no secure boot or FTM. These devices will follow different device identity and the same would be mentioned as part of exception flows.
* SBI 2.0 - A device can obtain SBI 2.0 certification when its built in secure facility with one of the certified FTM.

#### Device Identity
It is imperative that all devices that connect to MOSIP are identifiable. MOSIP believes in cryptographic Identity as its basis for trust.

##### Physical ID
An identification mark that shows MOSIP compliance and a readable unique device serial number (minimum of 12 digits), make and model. The same information has to be available over a 2D QR Code or Barcode. This is to help field support and validation.

##### Digital ID
A digital device ID in MOSIP would be a signed JSON (RFC 7515) as follows:
```JSON
{
  "serialNo": "Serial number",
  "make": "Make of the device",
  "model": "Model of the device",
  "type": "Type of the biometric device",
  "deviceSubType": "Subtypes of the biometric device",
  "deviceProvider": "Device provider name",
  "deviceProviderId": "Device provider id",
  "dateTime": "Datetime in ISO format with timezone. Identity request time"
}
```

Signed with the JSON Web Signature (RFC 7515) using the "Foundational Trust Module" Identity key, this data is the fundamental identity of the device. Every MOSIP compliant device will need the foundational trust module.

The only exception to this rule is for the SBI 1.0 compliant devices that have the purpose (explained below during device registration) as "Registration". SBI 1.0 devices would sign the digital Id with the device key.

Signed Digital ID would look as follows:
```
"digitalId": "base64urlencoded(header).base64urlencoded(payload).base64urlencoded(signature)"
```

The header in the digital id would have:
```
"alg": "RS256",
"type": "JWT",
"x5c": "<Certificate of the FTM chip, If in case the chain of certificates are sent then the same will be ignored">
```

MOSIP assumes that the first certificate in the x5c is the FTM's chip public certificate issued by the FTM root certificate.

Unsigned digital ID would look as follows:
```
"digitalId": "base64urlencoded(payload)"
```
Payload is the Digital ID JSON object.

{% hint style="info" %}
For a SBI 1.0 unregistered device the digital id will be unsigned. In all other scenarios, except for a discovery call, the digital ID will be signed either by the FTM key (SBI 2.0) or the device key (SBI 1.0).
{% endhint %}

##### Allowed Values
Parameters | Description
-----------|-------------
serialNo | This represents the serial number of the device. This value should be same as printed on the device (Refer [Physical ID](#physical-id)).
make | This represents the brand name of the device. This value should be same as printed on the device (Refer [Physical ID](#physical-id)).
model | This represents the model of the device. This value should be same as printed on the device (Refer [Physical ID](#physical-id)).
type | This represets the type of the device. Currently, the allowed values here are "Finger", "Iris" or "Face". More types can be added based on adopter's implementation.
deviceSubType | This represnts the device Sub type is based on the device type.<br><ul><li>For Finger - "Slap", "Single" or "Touchless"</li><li>For Iris - "Single" or "Double"</li><li>For Face - "Full face"</li></ul>
deviceProvider | This represnets the name of the device provider. The device provider should be a legal entity in the country.
dateTime | This contains the time during the issuance of an identity. This is in ISO format with timezone.

### Keys
List of keys used in the device and their explanation.

#### Device Key
Each biometric device would contain an authorized private key after the device registration. This key is rotated frequently based on the requirement from the respective adopter of MOSIP. By default MOSIP recommends 30 days key rotation policy for the device key. The device keys are created by the device providers inside the FTM during a successful registration.  The device keys are used for signing the biometric. More details of the signing and its usage will be [here](#device-service-communication-interfaces).  This key is issued by the device provider and the certificate of the device key is issued by the device provider key which in turn is issued by the MOSIP adopter after approval of the device providers specific model.

#### FTM Key
The FTM key is the root of the identity. This key is created by the FTM provider during the manufacturing/provisioning stage. This is a permanent key and would never be rotated. This key is used to sign the Digital ID.

#### MOSIP Key
The MOSIP key is the public key provided by the MOSIP adopter. This key is used to encrypt the biometric biometric. Details of the encryption is listed below. We recommend to rotate this key every 1 year.

## Device Service - Communication Interfaces
The section explains the necessary details of the biometric device connectivity, accessibility, discover-ability and protocols used to build and communicate with the device.

The device should implement only the following set of APIs.  All the API’s are independent of the physical layer and the operating system, with the invocation being different across operating systems. While the operating system names are defined in this spec a similar technology can be used for unspecified operating systems.  It is expected that the device service ensures that the device is connected  locally to the host.

### Device Discovery
Device discovery would be used to identify MOSIP compliant devices in a system by the applications. The protocol is designed as simple plug and play with all the necessary abstraction to the specifics.

#### Device Discovery Request
```JSON
{
  "type": "type of the device"
}
```

#### Allowed Values
Parameters | Description
-----------|-------------
type | This represents the type of device. Allowed values here are "Biometric Device", "Finger", "Face" or "Iris". "Biometric Device" - is a special type and used in case if you are looking for any biometric device.

#### Device Discovery Response
```JSON
[
  {
    "deviceId": "Internal ID",
    "deviceStatus": "Device status",
    "certification": "Certification level",
    "serviceVersion": "Device service version",
    "deviceSubId": ["Array of supported device sub Ids"],
    "callbackId": "Base URL to reach to the device",
    "digitalId": "Unsigned Digital ID of the device",
    "deviceCode": "A unique code given by MOSIP after successful registration",
    "specVersion": ["Array of supported SBI specification version"],
    "purpose": "Auth  or Registration or empty if not registered",
    "error": {
      "errorCode": "101",
      "errorInfo": "Invalid JSON Value Type For Discovery.."
    }
  },
  ...
]
```

#### Allowed Values
Parameters | Description
-----------|-------------
deviceStatus | Allowed values are "Ready", "Busy", "Not Ready" or "Not Registered".
certification | Allowed values are "SBI 1.0" or "SBI 2.0" based on level of certification.
serviceVersion | Version of the SBI specification that is supported.
deviceId | Internal ID to identify the actual biometric device within the device service.
deviceSubId | Allowed values are 1, 2 or 3. The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement. Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present. In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises. The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).
callbackId | This differs as per the OS. In case of Linux and windows operating systems it is a HTTP URL. In the case of android, it is the intent name. In IOS, it is the URL scheme. The call back URL takes precedence over future request as a base URL.
digitalId | Digital ID as per the Digital ID definition but it will not be signed.
deviceCode | A unique code given by MOSIP after successful registration.
specVersion | Array of supported SBI specification version.
purpose | Purpose of the device in the MOSIP ecosystem. Allowed values are "Auth" or "Registration".
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code.
error.errorInfo | Description of the error that can be displayed to end user. It should have multi-lingual support.

{% hint style="info" %}
* The response is an array that we could have a single device enumerating with multiple biometric options.
* The service should ensure to respond only if the type parameter matches the type of device or the type parameter is a "Biometric Device".
* This response is a direct JSON as show in the response.
{% endhint %}

#### Windows/Linux
All the device API will be based on the HTTP specification. The device always binds to with any of the available ports ranging from 4501 - 4600.  The IP address used for binding has to be 127.0.0.1 and not localhost.

The applications that require access to MOSIP devices could discover them by sending the HTTP request to the supported port range. We will call this port as the device_service_port in the rest of the document.

**_HTTP Request:_**
```
MOSIPDISC http://127.0.0.1:<device_service_port>/device
HOST: 127.0.0.1: <device_service_port>
EXT: <app name>
```

**_HTTP Response:_**
```
HTTP/1.1 200 OK
CACHE-CONTROL:no-store
LOCATION:http://127.0.0.1:<device_service_port>
Content-Length: length in bytes of the body
Content-Type: application/json
Connection: Closed
```

{% hint style="info" %}
* The pay loads are JSON in both the cases and are part of the body.
* CallbackId would be set to the `http://127.0.0.1:<device_service_port>`. So, the caller will use the respective HTTP verb / method and the URL to call the service.
{% endhint %}

#### Android
All devices on an android device should listen to the following intent "io.mosip.device".

Upon invocation of this intent the devices are expected to respond back with the json response filtered by the respective type.

{% hint style="info" %}
In Android, the CallbackId would be set to the appId. So, the caller will create the intent "appId.Info" or "appId.Capture".
{% endhint %}

#### IOS
All device on an IOS device would respond to the URL schema as follows:

```
MOSIPDISC://<call-back-app-url>?ext=<caller app name>&type=<type as defined in mosip device request>
```

If a MOSIP compliant device service app exist then the URL would launch the service. The service in return should respond back to the caller using the call-back-app-url with the base64 encoded json as the URL parameter for the key data.

{% hint style="info" %}
* In IOS there are restrictions to have multiple apps registering to the same URL schema.
* CallbackId would be set to the device service appname. So, the caller has to call appnameInfo or appnameCapture as the URL scheme.
{% endhint %}

### Device Info
The device information API would be used to identify the MOSIP compliant devices and their status by the applications.

#### Device Info Request
NA

#### Allowed Values
NA

#### Device Info Response
```
[
  {
    "deviceInfo": {
      "deviceStatus": "Current status",
      "deviceId": "Internal ID",
      "firmware": "Firmware version",
      "certification": "Certification level",
      "serviceVersion": "Device service version",
      "deviceSubId": ["Array of supported device sub Ids"],
      "callbackId": "BaseURL to reach to the device",
      "digitalId": "Signed digital id as described in the digital id section of this document.",
      "deviceCode": "A unique code given by MOSIP after successful registration",
      "env": "Target environment",
      "purpose": "Auth  or Registration",
      "specVersion": ["Array of supported SBI specification version"],
    },
    "error": {
      "errorCode": "101",
      "errorInfo": "Invalid JSON Value "
    }
  }
  ...
]
```

The final JSON is signed with the JSON Web Signature using the "Foundational Trust Module" Identity key, this data is the fundamental identity of the device. Every MOSIP compliant device will need the foundational trust module.

So the API would respond in the following format.
```
[
  {
    "deviceInfo": "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
    "error": {
      "errorCode": "100",
      "errorInfo": "Device not registered. In this case the device info will be only base64urlencode(payload)"
    }
  }
]
```

#### Allowed Values
Parameters | Description
-----------|-------------
deviceInfo | The deviceInfo object is sent as JSON Web Token (JWT). For device which is not registered, the deviceInfo will be unsigned. For device which is registered, the deviceInfo will be signed using the device key.
deviceInfo.deviceStatus | This is the current status of the device. Allowed values are "Ready", "Busy", "Not Ready" or "Not Registered".
deviceInfo.deviceId | Internal Id to identify the actual biometric device within the device service.
deviceInfo.firmware | Exact version of the firmware. In case of SBI 1.0 this is same as serviceVersion.
deviceInfo.certification | Allowed values are "SBI 1.0" or "SBI 2.0" based on the level of certification.
deviceInfo.serviceVersion | Version of the SBI specification that is supported.
deviceInfo.deviceId | Internal ID to identify the actual biometric device within the device service.
deviceSubId | Allowed values are 1, 2 or 3. The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement. Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present. In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises. The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).
deviceInfo.callbackId | This differs as per the OS. In case of linux and windows operating systems it is a HTTP URL. In the case of android, it is the intent name. In IOS, it is the URL scheme. The call back URL takes precedence over future request as a base URL.
deviceInfo.digitalId | The digital id as per the digital id definition. For SBI 1.0 devices which is not registered, the digital id will be unsigned. For SBI 1.0 devices which is registered, the digital id will be signed using the device key. For SBI 2.0 devices, the digital id will be signed using the FTM key.
deviceInfo.env | This represents the target enviornment. For devices that are not registered the enviornment is "None". For device that is registered, then send the enviornment in which it is registered. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".
deviceInfo.purpose | The purpose of the device in the MOSIP ecosystem. For devices that are not registered the purpose is empty. Allowed values are "Auth" or "Registration".
deviceInfo.specVersion | Array of supported SBI specification version.
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code.
error.errorInfo | Description of the error that can be displayed to end user. It should have multi-lingual support.

{% hint style="info" %}
* The response is an array that we could have a single device enumerating with multiple biometric options.
* The service should ensure to respond only if the type parameter matches the type of device or the type parameter is a "Biometric Device".
{% endhint %}

#### Windows/Linux
The applications that require more details of the MOSIP devices could get them by sending the HTTP request to the supported port range. The device always binds to with any of the available ports ranging from 4501 - 4600. The IP address used for binding has to be 127.0.0.1 and not localhost.

**_HTTP Request:_**
```
MOSIPDINFO http://127.0.0.1:<device_service_port>/info
HOST: 127.0.0.1:<device_service_port>
EXT: <app name>
```

**_HTTP Response:_**
```
HTTP/1.1 200 OK
CACHE-CONTROL:no-store
LOCATION:http://127.0.0.1:<device_service_port>
Content-Length: length in bytes of the body
Content-Type: application/json
Connection: Closed
```

{% hint style="info" %}
The pay loads are JSON in both the cases and are part of the body.
{% endhint %}

#### Android
On an android device should listen to the following intent "appId.Info".

Upon invocation of this intent the devices are expected to respond back with the JSON response filtered by the respective type.

#### IOS
On an IOS device would respond to the URL schema as follows:
```
APPIDINFO://<call-back-app-url>?ext=<caller app name>&type=<type as defined in mosip device request>
```

If a MOSIP compliant device service app exist then the URL would launch the service. The service in return should respond back to the called using the call-back-app-url with the base64 encoded JSON as the URL parameter for the key data.

{% hint style="info" %}
In IOS there are restrictions to have multiple app registering to the same URL schema.
{% endhint %}

### Capture
The capture request would be used to capture a biometric from MOSIP compliant devices by the applications. The capture call will respond with success to only one call at a time. So, in case of a parallel call the device info details are sent with status as "Busy".

#### Capture Request
```
{
  "env": "Target environment",
  "purpose": "Auth  or Registration",
  "specVersion": "Expected version of the SBI spec",
  "timeout" : "Timeout for capture",
  "captureTime": "Time of capture request in ISO format including timezone",
  "domainUri": "URI of the auth server",
  "transactionId": "Transaction Id for the current capture",
  "bio": [
    {
      "type": "Type of the biometric data",
      "count":  "Finger/Iris count, in case of face max is set to 1",
      "bioSubType": ["Array of subtypes"],
      "requestedScore": "Expected quality score that should match to complete a successful capture",
      "deviceId": "Internal Id",
      "deviceSubId": "Specific Device Sub Id",
      "previousHash": "Hash of the previous block"
    }
  ],
  customOpts: {
	//max of 50 key value pair.
	//This is so that vendor specific parameters can be sent if necessary.
	//The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications.
	//Vendors are free to include additional parameters and fine-tuning parameters.
	//None of these values should go undocumented by the vendor.
	//No sensitive data should be available in the customOpts.
  }
}
```

{% hint style="info" %}
Count value should be driven by the count of the bioSubType for Iris and Finger. For Face, there will be no bioSubType but the count should be "1".
{% endhint %}

#### Allowed Values
Parameters | Description
-----------|-------------
env | This represents the target environment. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".
purpose | The purpose of the device in the MOSIP ecosystem. For devices that are not registered the purpose is empty. Allowed values are "Auth" or "Registration".
specVersion | Expected version of SBI specification.
timeout | Max time the app will wait for the capture. Its expected that the API will respond back before timeout with the best frame. All timeouts are in milliseconds.
captureTime | Time of capture in ISO format with timezone. The time is as per the requesting application.
domainUri | URI of the authentication server. This can be used to federate across multiple providers or countries or unions.
transactionId | Unique ID for the transaction. This is an internal Id to the application thats providing the service. Different id should be used for every transaction. So, even if the transaction fails after auth we expect this number to be unique.
bio.type | Allowed values are "Finger", "Iris" or "Face".
bio.count | Number of biometric data that is collected for a given type. The device should validate and ensure that this number is in line with the type of biometric that's captured.
bio.bioSubType | <ul><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li></ul>
bio.requestedScore | Upon reaching the quality score the biometric device is expected to auto capture the image.
bio.deviceId | Internal Id to identify the actual biometric device within the device service.
bio.deviceSubId | Allowed values are 1, 2 or 3. The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement. Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present. In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises. The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).
bio.previousHash | For the first capture the previousHash is hash of empty UTF-8 string. From the second capture the previous captures hash (as hex encoded) is used as input. This is used to chain all the captures across modalities so all captures have happened for the same transaction and during the same time period.
customOpts | In case, the device vendor wants to send additional parameters they can use this to send key value pair if necessary. The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications. Vendors are free to include additional parameters and fine-tuning the process. None of these values should go undocumented by the vendor. No sensitive data should be available in the customOpts.

#### Capture Response
```
{
  "biometrics": [
    {
      "specVersion": "SBI spec version",
      "data": { //data block in JWT format signed using device key
        "digitalId": "digital Id as described in this document signed using FTM chip key",
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "deviceServiceVersion": "SBI version",
        "bioType": "Finger",
        "bioSubType": "UNKNOWN",
        "purpose": "Auth  or Registration",
        "env": "Target environment",
        "domainUri": "URI of the auth server",
        "bioValue": "Encrypted with session key and base64urlencoded biometric data",
        "transactionId": "Unique transaction id",
        "timestamp": "ISO format datetime with time zone",
        "requestedScore": "Floating point number to represent the minimum required score for the capture",
        "qualityScore": "Floating point number representing the score for the current capture"
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block before encryption)",
      "sessionKey": "encrypted with MOSIP public key (dynamically selected based on the uri) and encoded session key biometric",
      "thumbprint": "SHA256 representation of thumbprint of the certificate that was used for encryption of session key. All texts to be treated as uppercase without any spaces or hyphens",
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value"
      }
    },
    {
      "specVersion" : "SBI spec version",
      "data": { //data block in JWT format signed using device key
        "digitalId": "Digital Id as described in this document signed using FTM chip key",
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "deviceServiceVersion": "SBI version",
        "bioType": "Finger",
        "bioSubType": "Left IndexFinger",
        "purpose": "Auth  or Registration",
        "env": "Target environment",
        "domainUri": "URI of the auth server",
        "bioValue": "Encrypted with session key and base64urlencoded biometric data",
        "transactionId": "Unique transaction id",
        "timestamp": "ISO Format date time with timezone",
        "requestedScore": "Floating point number to represent the minimum required score for the capture",
        "qualityScore": "Floating point number representing the score for the current capture"
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block before encryption)",
      "sessionKey": "encrypted with MOSIP public key and encoded session key biometric",
      "thumbprint": "SHA256 representation of thumbprint of the certificate that was used for encryption of session key. All texts to be treated as uppercase without any spaces or hyphens",
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value"
      }
    }
  ]
}
```

#### Allowed Values
Parameters | Description
-----------|------------- 
specVersion | Version of the SBI specification using which the response was generated.
data | The data object is sent as JSON Web Token (JWT). The data block will be signed using the device key.
data.digitalId | The digital id as per the digital id definition in JWT format. For SBI 1.0 devices, the digital id will be signed using the device key. For SBI 2.0 devices, the digital id will be signed using the FTM key.
data.deviceCode | A unique code given by MOSIP after successful registration
data.deviceServiceVersion | SBI version
data.bioType | Allowed values are "Finger", "Iris" or "Face".
data.bioSubType | <ul><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li></ul>
data.purpose | The purpose of the device in the MOSIP ecosystem. Allowed values is "Auth".
data.env | The target environment. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".
data.domainUri | URI of the authentication server. This can be used to federate across multiple providers or countries or unions.
data.bioValue | Biometric data is encrypted with random symmetric (AES GCM) key and base-64-URL encoded. For symmetric key encryption of bioValue, (biometrics.data.timestamp XOR transactoinId) is computed and the last 16 bytes and the last 12  bytes of the results are set as the aad and the IV(salt) respectively. Look at the Authentication document to understand more about the encryption.
data.transactionId | Unique transaction id sent in request
data.timestamp | Time as per the biometric device. Note: The biometric device is expected to sync its time from the management server at regular intervals so accurate time could be maintained on the device.
data.requestedScore | Floating point number to represent the minimum required score for the capture.
data.qualityScore | Floating point number representing the score for the current capture.
hash | The value of the previousHash attribute in the request object or the value of hash attribute of the previous data block (used to chain every single data block) concatenated with the hex encode sha256 hash of the current data block before encryption.
sessionKey | The session key (used for the encrypting of the bioValue) is encrypted using the MOSIP public certificate with RSA/ECB/OAEPWITHSHA-256ANDMGF1PADDING algorithm and then base64-URL-encoded.
thumbprint | SHA256 representation of thumbprint of the certificate that was used for encryption of session key. All texts to be treated as uppercase without any spaces or hyphens.
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code.
error.errorInfo | Description of the error that can be displayed to end user. It should have multi-lingual support.

The entire data object is sent as a JWT format. So, the data object will look like:
```
"data" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)
```
The payload is defined as the entire byte array of data block.


#### Windows/Linux
The applications that requires to capture biometric data from a MOSIP devices could do so by sending the HTTP request to the supported port range.

**_HTTP Request:_**
```
CAPTURE [http://127.0.0.1:<device_service_port>/capture](http://127.0.0.1/capture)
HOST: 127.0.0.1: <apps port>
EXT: <app name>
```

**_HTTP Response:_**
```
HTTP/1.1 200 OK
CACHE-CONTROL:no-store
LOCATION:[http://127.0.0.1](http://127.0.0.1):<device_service_port>
Content-Length: length in bytes of the body
Content-Type: application/json
Connection: Closed
```

{% hint style="info" %}
The pay loads are JSON in both the cases and are part of the body.
{% endhint %}

#### Android
All device on an android device should listen to the following intent appid.capture.  Upon this intend the devices are expected to respond back with the JSON response filtered by the respective type.

#### IOS
All device on an IOS device would respond to the URL schema as follows.

```
APPIDCAPTURE://<call-back-app-url>?ext=<caller app name>&type=<type as defined in mosip device request>
```

If a MOSIP compliant device service app exist then the URL would launch the service. The service in return should respond back to the called using the call-back-app-url with the base64 encoded json as the URL parameter for the key data.

### Device Stream
The device would open a stream channel to send the live video streams. This would help when there is an assisted operation to collect biometric. Please note the stream API’s are available only for registration environment.

Used only for the registration module compatible devices. This API is visible only for the devices that are registered for the purpose as "Registration".

#### Device Stream Request
```
{
  "deviceId": "Internal Id",
  "deviceSubId": "Specific device sub Id",
  "timeout": "Timeout for stream"
}
```

#### Allowed Values
Parameters | Description
-----------|--------------
deviceId | Internal Id to identify the actual biometric device within the device service.
deviceSubId | Allowed values are 1, 2 or 3. The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement. Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present. In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises. The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).
timeout | Max time after which the stream should close. This is an optional paramter and by default the value will be 5 minutes. All timeouts are in milliseconds.

#### Device Stream Response
Live video stream with quality of 3 frames per second or more using [M-JPEG](https://en.wikipedia.org/wiki/Motion_JPEG).

{% hint style="info" %}
Preview should have the quality markings and segment marking. The preview would also be used to display any error message to the user screen. All error messages should be localized.
{% endhint %}

#### Error Response for Device Stream
```
{
  "error": {
    "errorCode": "202",
    "errorInfo": "No Device Connected."
  }
}
```

#### Windows/Linux
The applications that require more details of the MOSIP devices could get them by sending the HTTP request to the supported port range.

**_HTTP Request:_**
```
STREAM   http://127.0.0.1:<device_service_port>/stream
HOST: 127.0.0.1: <apps port>
EXT: <app name>
```

**_HTTP Response:_**
HTTP Chunk of frames to be displayed. Minimum frames 3 per second.

#### Android
No support for streaming

#### IOS
No support for streaming

### Registration Capture
The registration client application will discover the device. Once the device is discovered the status of the device is obtained with the device info API. During the registration the registration client sends the RCAPTURE API and the response will provide the actual biometric data in a digitally signed non encrypted form. When the Device Registration Capture API is called the frames should not be added to the stream. The device is expected to send the images in ISO format.

The requestedScore is in the scale of 1-100. So, in cases where you have four fingers the average of all will be considered for capture threshold. The device would always send the best frame during the capture time even if the requested score is not met.

The API is used by the devices that are compatible for the registration module. This API should not be supported by the devices that are compatible for authentication.

#### Registration Capture Request
```
{
  "env":  "Target environment",
  "purpose": "Auth  or Registration",
  "specVersion": "Expected SBI spec version",
  "timeout": "Timeout for registration capture",
  "captureTime": "Time of capture request in ISO format including timezone",
  "transactionId": "Transaction Id for the current capture",
  "bio": [
    {
      "type": "Type of the biometric data",
      "count":  "Finger/Iris count, in case of face max is set to 1",
      "bioSubType": ["Array of subtypes"], //Optional
      "exception": ["Finger or Iris to be excluded"],
      "requestedScore": "Expected quality score that should match to complete a successful capture.",
      "deviceId": "Internal Id",
      "deviceSubId": "Specific device Id",
      "previousHash": "Hash of the previous block"
    }
  ],
  customOpts: {
    //max of 50 key value pair.
	//This is so that vendor specific parameters can be sent if necessary.
	//The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications.
	//Vendors are free to include additional parameters and fine-tuning parameters.
	//None of these values should go undocumented by the vendor.
	//No sensitive data should be available in the customOpts.
  }
}
```

#### Allowed Values
Parameters | Description
-----------|-------------
env | The target environment. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".
purpose | The purpose of the device in the MOSIP ecosystem. For devices that are not registered the purpose is empty.</li><li>Allowed values are "Auth" or "Registration".
specVersion | Expected version of SBI specification.
timeout | Max time the app will wait for the capture. It is expected that the API will respond back before timeout with the best frame. All timeouts are in milliseconds.
captureTime | Time of capture in ISO format with timezone. The time is as per the requesting application.
transactionId | Unique ID of the transaction. This is an internal Id to the application thats providing the service. Different id should be used for every transaction. So, even if the transaction fails we expect this number to be unique.
bio.type | Allowed values are "Finger", "Iris" or "Face".
bio.count | Number of biometric data that is collected for a given type. The device should validate and ensure that this number is in line with the type of biometric that's captured.
bio.bioSubType | <ul><li>Array of bioSubType for respective biometric type.</li><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li><li>This is an optional parameter.</li></ul>
bio.exception | <ul><li>This is an array and all the exceptions are marked.</li><li>In case exceptions are sent for face then follow the exception photo specification above.</li><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb"]</li><li>For Iris: ["Left", "Right"]</li></ul>
bio.requestedScore | Upon reaching the quality score the biometric device is expected to auto capture the image.
bio.deviceId | Internal Id to identify the actual biometric device within the device service.
bio.deviceSubId | Allowed values are 1, 2 or 3. The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement. Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present. In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises. The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).
bio.previousHash | For the first capture the previousHash is hash of empty UTF-8 string. From the second capture the previous captures hash (as hex encoded) is used as input. This is used to chain all the captures across modalities so all captures have happened for the same transaction and during the same time period.
customOpts | In case, the device vendor wants to send additional parameters they can use this to send key value pair if necessary. The values cannot be hard coded and have to be configured by the apps server and should be modifiable upon need by the applications. Vendors are free to include additional parameters and fine-tuning the process. None of these values should go undocumented by the vendor. No sensitive data should be available in the customOpts.

#### Registration Capture Response
```
{
  "biometrics": [
    {
      "specVersion": "SBI Spec version",
      "data": {
        "digitalId": "Digital id of the device as per the Digital Id definition..",
        "bioType": "Biometric type",
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "deviceServiceVersion": "SBI version",
        "bioSubType": "Left IndexFinger",
        "purpose": "Auth  or Registration",
        "env": "Target environment",
        "bioValue": "base64urlencoded biometrics (ISO format)",
        "transactionId": "Unique transaction id sent in request",
        "timestamp": "2019-02-15T10:01:57.086+05:30",
        "requestedScore": "Floating point number to represent the minimum required score for the capture. This ranges from 0-100.",
        "qualityScore": "Floating point number representing the score for the current capture. This ranges from 0-100."
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block)",    
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value Type For Discovery.. ex: {type: 'Biometric Device' or 'Finger' or 'Face' or 'Iris' } "
      }
    },
    {
      "specVersion" : "SBI Spec version",
      "data": {
        "deviceCode": "A unique code given by MOSIP after successful registration",
        "bioType" : "Finger",
        "digitalId": "Digital id of the device as per the Digital Id definition.",
        "deviceServiceVersion": "SBI version",
        "bioSubType": "Left MiddleFinger",
        "purpose": "Auth  or Registration",
        "env":  "Target environment",
        "bioValue": "base64urlencoded extracted biometric (ISO format)",
        "transactionId": "Unique transaction id sent in request",
        "timestamp": "2019-02-15T10:01:57.086+05:30",
        "requestedScore": "Floating point number to represent the minimum required score for the capture. This ranges from 0-100",
        "qualityScore": "Floating point number representing the score for the current capture. This ranges from 0-100"
      },
      "hash": "sha256(sha256 hash in hex format of the previous data block + sha256 hash in hex format of the current data block before encryption)",
      "error": {
        "errorCode": "101",
        "errorInfo": "Invalid JSON Value Type For Discovery.. ex: {type: 'Biometric Device' or 'Finger' or 'Face' or 'Iris' }"
      }
    }
  ]
}
```

#### Allowed Values
Parameters | Description
-----------|-------------
specVersion | Version of the SBI specification using which the response was generated.
data | The data object is sent as JSON Web Token (JWT). The data block will be signed using the device key.
data.bioType | Allowed values are "Finger", "Iris" or "Face".
data.digitalId | The digital id as per the digital id definition in JWT format. For SBI 1.0 devices, the digital id will be signed using the device key. For SBI 2.0 devices, the digital id will be signed using the FTM key.
data.bioSubType | <ul><li>For Finger: ["Left IndexFinger", "Left MiddleFinger", "Left RingFinger", "Left LittleFinger", "Left Thumb", "Right IndexFinger", "Right MiddleFinger", "Right RingFinger", "Right LittleFinger", "Right Thumb", "UNKNOWN"]</li><li>For Iris: ["Left", "Right", "UNKNOWN"]</li><li>For Face: No bioSubType</li></ul>
data.deviceServiceVersion | SBI version
data.env | The target environment. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".
data.purpose | The purpose of the device in the MOSIP ecosystem. Allowed values are "Auth" or "Registration".
data.bioValue | Base64-URL-encoded biometrics (in ISO format)
data.transactionId | Unique transaction id sent in request
data.timestamp | Time as per the biometric device. Note: The biometric device is expected to sync its time from the management server at regular intervals so accurate time could be maintained on the device.
data.requestedScore | Floating point number to represent the minimum required score for the capture.
data.qualityScore | Floating point number representing the score for the current capture.
hash | The value of the previousHash attribute in the request object or the value of hash attribute of the previous data block (used to chain every single data block) concatenated with the hex encode sha256 hash of the current data block before encryption.
error | Relevant errors as defined under the [error section](#error-codes) of this document.
error.errorCode | Standardized error code.
error.errorInfo | Description of the error that can be displayed to end user. It should have multi-lingual support..

#### Windows/Linux
The applications that require more details of the MOSIP devices could get them by sending the HTTP request to the supported port range.

**_HTTP Request:_**
```
RCAPTURE  http://127.0.0.1:<device_service_port>/capture
HOST: 127.0.0.1: <apps port>
EXT: <app name>
```

**_HTTP Response:_**
HTTP response.

#### Android
No support for Registration Capture

#### IOS
No support for Registration Capture

---

## Device Server
The device server exposes two external device APIs to manage devices. These will be consumed from Management Server created by the device provider. Refer to the subsequent section in this document.

### Registration
The MOSIP server would provide the following device registration API which is white-listed to the management servers of the device provider or their partners.

{% hint style="info" %}
This API is exposed by the MOSIP server to the device providers.
{% endhint %}

**Version:** v1

#### Device Registration Request URL
`POST https://{base_url}/v1/masterdata/registereddevices`

#### Device Registration Request
```
{
  "id": "io.mosip.deviceregister",
  "request": {
    "deviceData": {
      "deviceId": "Unique Id to identify a biometric capture device",
      "purpose": "Auth or Registration. Can not be empty.",
      "deviceInfo": {
        "deviceSubId": "An array of sub Ids that are available",
        "certification":  "certification level",
        "digitalId": "Signed digital id of the device",
        "firmware": "Firmware version",
        "deviceExpiry": "Device expiry date",
        "timestamp":  "ISO format datetime with timezone from device"
      },
      "foundationalTrustProviderId" : "Foundation trust provider Id, in case of SBI 1.0 this is empty"
    }
  },
  "requesttime": "Current timestamp in ISO format from management server",
  "version": "Registration server API version as defined above"
}
```

#### Allowed Values
Parameters | Description
-----------|-------------
deviceData | The device data object is sent as JSON Web Token (JWT). The device data block will be signed using the device provider certificate.
deviceData.deviceId | Unique device id that the device provider uses to identify the device. This can also be serial no if the device provider is sure of maintaining the uniqueness across all their devices.
purpose | The purpose of the device in the MOSIP ecosystem. For devices that are not registered the purpose is empty. Allowed values are "Auth" or "Registration".
deviceData.deviceInfo | The device info object is sent as JSON Web Token (JWT). The device info block will be signed using the device key.
deviceInfo.deviceSubId | An array of sub Ids that are supported for the device. Allowed values are 1, 2 or 3. The device sub id could be used to enable a specific module in the scanner appropriate for a biometric capture requirement. Device sub id is a simple index which always starts with 1 and increases sequentially for each sub device present. In case of Finger/Iris its 1 for left slap/iris, 2 for right slap/iris and 3 for two thumbs/irises. The device sub id should be set to 0 if we don't know any specific device sub id (0 is not applicable for fingerprint slap).
deviceInfo.certification | The certificate level of the device. Allowed values are SBI 1.0 or SBI 2.0.
deviceInfo.digitalId | The digital id as per the digital id definition. For SBI 1.0 devices, the digital id will be signed using the device key. For SBI 2.0 devices, the digital id will be signed using the FTM key.
deviceInfo.firmware | Version of the firmware of the device.
deviceInfo.deviceExpiry | Expiry date of the device. Device will not work post that expiry date and it cannot be registered again.
deviceInfo.timestamp | The timestamp when the request was created. For device registration the request should reach within 5 mins of this timestamp to MOSIP or the request will be rejected.
foundationalTrustProviderId | Foundation trust provider id whoes chip is in the device. For SBI 1.0 device, this is empty.

{% hint style="info" %}
During the registration of SBI 1.0 devices please sign using the key thats generated inside the device and send the public key in x509 encoded spec form. <br>
After successful registration the management server should issue a certificate against the same public key as a response to the registration call.
{% endhint %}

* The entire device data is sent as a JWT format. So the it will look like:
```
"deviceData" : base64urlencode(header).base64urlencode(payload).base64urlencode(signature)
```
* Payload is the object in deviceData.
* The request is signed with the device provider key at the management server.

#### Device Registration Response
```
{
  "id": "io.mosip.deviceregister",
  "version": "Registration server API version as defined above",
  "responsetime": "ISO time format",
  "response": {
    "status":  "Registration status",
    "digitalId": "Digital id of the device a sent by the request",
    "deviceCode": "UUID RFC4122 Version 4 for the device issued by the mosip server",
    "timestamp": "Timestamp in ISO format",
    "env": "Production/Developer/Staging/Pre-Production"
  },
  "error": [
    {
      "errorCode": "Error code if registration fails. Remaining keys above are dropped in case of errors.",
      "message": "Description of the error code"
    }
  ]
}
```

The entire response is sent as a JWT format. So the final response will look like:
```
"response" : base64urlencode(header).base64urlencode(payload).base64urlencode(signature)
```

### Accepted Values
Parameters | Description
-----------|-------------
response | The entire response block will be sent in JWT format. This will be signed by MOSIP using their public signature certificate.
response.status | This is the status of the device. After successful registration the status will sent as "Registered".
response.digitalId | This is the same digital ID that was sent in the request.
response.deviceCode | This is the device code issued by MOSIP server post registration. This will be in UUID RFC4122 Version 4 format. Once device is registered the device code needs to be set in the device.
response.timestamp | This is the timestamp when the device was registered. This will be in ISO format.
response.env | The target environment where the device is registered. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".

The response should be sent to the device. The device is expected to store the deviceCode within its storage in a safe manner. This device code is used during the capture stage.

{% hint style="info" %}
The device once registered for a specific purpose can not be changed after successful registration. The device can only be used for that specific MOSIP process.
{% endhint %}

### De-Register
The MOSIP server would provide the following device de-registration API which is whitelisted to the management servers of the device provider or their partners.

**Version:** v1

#### Device De-Registration Request URL
`POST https://{base_url}/v1/masterdata/device/deregister`

#### Device De-Registration Request
```
{
  "id": "io.mosip.devicederegister",
  "version": "de-registration server api version as defined above",
  "request": {
    "device": {
      "deviceCode": "<device code>",
      "env": "<environment>"
    }
  }
  "requesttime": "current timestamp in ISO format"
}
```

The device data in request is sent as a JWT format. So the final request will look like:
```
"request": {
  "device" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
}
```

##### Allowed Values
Parameters | Description
-----------|-------------
request.device | The device object is sent as JSON Web Token (JWT). The device block will be signed using the device provider certificate.
request.device.deviceCode | This is the device code issued by MOSIP server during device registration, which needs to be de-registered.
request.device.env | The target environment where the device is registered. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".

#### Device De-Registration Response
```
{
  "id": "io.mosip.devicederegister",
  "version": "de-registration server api version as defined above",
  "responsetime": "iso time format",
  "response": {
    "status": "Success",
    "deviceCode": "<device code>",
    "env": "<environment>",
    "timestamp": "timestamp in ISO format"
  },
  "error": [
        {
      "errorCode" : "<error code if de-registration fails>",
      "message" : "<human readable description of the error code>"
    }
  ]
}
```

The entire response is sent as a JWT format. So the final response will look like:

```
"response" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
```

##### Allowed Values
Parameters | Description
-----------|-------------
response | The entire response block will be sent in JWT format. This will be signed by MOSIP using their public signature certificate.
response.status | The status of de-registration request. The status will be "Success" for successful de-registration.
response.deviceCOde | This is the deviceCode for the device that got de-registered.
response.env | The target environment where the device is de-registered. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".
response.timestamp | The time when the device got de-registered. Timestamp is in ISO format.

---

### Certificates
The MOSIP server would provide the following retrieve encryption certificate API which is white-listed to the management servers of the device provider or their partners.

#### Retrieve Encryption Certificate Request URL
`POST https://{base_url}/v1/masterdata/device/encryptioncertficates`

**Version:** v1

#### Retrieve Encryption Certificate Request
```
{
  "id": "io.mosip.auth.country.certificate",
  "version": "certificate server api version as defined above",
  "request": {
    "data": {
      "env":  "target environment",
      "domainUri": "uri of the auth server"
    }
  },
  "requesttime": "current timestamp in ISO format"
}
```

The request is sent as a JWT format. So the final request will look like:
```
"request": {
  "data": "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
}
```

##### Allowed Values
Parameters | Description
-----------|-------------
request.data | The data object is sent as JSON Web Token (JWT).
request.data.env | The target environment for which you want to fetch the certificate. Allowed values are "Staging", "Developer", "Pre-Production" or "Production".
request.data.domainUri | unique URI per auth providers. This can be used to federate across multiple providers or countries or unions.

#### Encryption Certificate Response
```
{
  "id": "io.mosip.auth.country.certificate",
  "version": "certificate server api version as defined above",
  "responsetime": "iso time format",
  "response": [
    {
      "certificate": "base64encoded certificate as x509 V3 format"
    }
  ]
}
```

The entire response is sent as a JWT format. So the final response will look like:
```
"response" : "base64urlencode(header).base64urlencode(payload).base64urlencode(signature)"
```

---

## Management Server and Management Client

### Management Server Functionalities and Interactions
The management server has the following objectives.

1. Validate the devices to ensure its a genuine device from the respective device provider. This can be achieved using the device info and the certificates for the Foundational Trust Module.
1. Register the genuine device with the MOSIP device server.
1. Manage/Sync time between the end device the server. The time to be synced should be the only trusted time accepted by the device.
1. Ability to issue commands to the end device for
    1. De-registration of the device (Device Keys)
    1. Collect device information to maintain, manage, support and upgrade a device remotely.
1. A central repository of all the approved devices from the device provider.
1. Safe storage of keys using HSM FIPS 140-2 Level 3. These keys are used to issue the device certificate upon registration.
The Management Server is created and hosted by the device provider outside of MOSIP software. The communication protocols between the SBI and the Management Server can be decided by the respective device provider. Such communication should be restricted to the above specified interactions only. No transactional information should be sent to this server.
1. Should have the ability to push updates from the server to the client devices.

### Management Client
Management client is the interface that connects the device with the respective management server. Its important that the communication between the management server and its clients are designed with scalability, robustness, performance and security. The management server may add many more capabilities than what is described here, But the basic security objectives should be met at all times irrespective of the offerings.

1. For better and efficient handling of device at large volume, we expect the devices to auto register to the Management server.
1. All communication to the server and from the server should follow that below properties.
    1. All communication are digitally signed with the approved algorithms
    1. All communication to the server are encrypted using one of the approved public key cryptography (HTTPS – TLS1.2/1.3 is required with one of the [approved algorithms](#cryptography).
    1. All request has timestamps attached in ISO format to the milliseconds inside the signature.
    1. All communication back and fourth should have the signed digital id as one of the attribute.
1. Its expected that the auto registration has an absolute way to identify and validate the devices.
1. The management client should be able to detect the devices in a plug and play model.
1. All key rotation should be triggered from the server.
1. Should have the ability to detect if its speaking to the right management server.
1. All upgrades should be verifiable and the client should have ability to verify software upgrades.
1. Should not allow any downgrade of software.
1. Should not expose any API to capture biometric. The management server should have no ability to trigger a capture request.
1. No logging of biometric data is allowed. (Both in the encrypted and unencrypted format)

---

## Compliance
**SBI 1.0 Certified Device / SBI 1.0 Device** - A device certified as one where the encryption is done on the host inside its device driver or the Secure Biometric Interface.

**SBI 2.0 Certified Device / SBI 2.0 Device** - A device certified as capable of performing encryption on the device inside its trusted zone with tamper responsive features.

### Secure Provisioning
Secure provisioning is applicable to both the FTM and the Device providers.

1. The devices and FTM should have a mechanism to protect against fraudulent attempts to create or replicate.
1. The device and FTM trust should be programmed in a secure facility which is certified by the respective MOSIP adopters.
1. Organization should have mechanism to segregate the FTM's and Devices built for MOSIP using cryptographically valid and repeatable process.
1. All debug options within the FTM or device should be disabled permanently
1. All key creations need for provisioning should happen automatically using FIPS 140-2 Level 3 or higher devices. No individual or a group or organization should have mechanism to influence this behavior.
1. Before the devices/FTM leaving the secure provisioning facility all the necessary trust should be established and should not be re-programmable.

### Compliance Level
API     | Compatible
--------|-----------
Device Discovery | SBI 1.0 / SBI 2.0
Device Info | SBI 1.0 / SBI 2.0
Capture | SBI 2.0
Registration Capture | SBI 1.0 / SBI 2.0

---

## Cryptography
Supported algorithms:

Usage | Algorithm | Key Size | Storage
------|-----------|----------|---------
Encryption of biometrics - Session Key | AES | >=256 | No storage, Key is created with TRNG/DRBG inside FTM
Encryption session key data outside of FTM | RSA OAEP | >=2048 | FTM trusted memory
Encryption session key data outside of FTM | ECC curve 25519 | >=256 | FTM trusted memory
Biometric Signature | RSA | >=2048 | Key never leaves FTM created and destroyed
Biometric Signature | ECC curve 25519 | >=256 | Key never leaves FTM created and destroyed
Secure Boot | RSA | >=256 | FTM trusted memory
Secure Boot | ECC curve 25519 | >=256 | FTM trusted memory

{% hint style="info" %}
No other ECC curves supported.
{% endhint %}

## Signature
In all the above APIs, some of the requests and responses are signed with various keys to verify the requester's authenticity. Here we have detailed the key used for signing a particular block in a request or response body of various APIs.

Request/Response | Block | Signature Key
-----------------|-------|---------------
Device Discovery Response | Device Info | NA as it will not be signed
Device Discovery Response | Digital ID | NA as it will not be signed
Device Info Response | Device Info | <ul><li>NA in case of unregistered device</li><li>Device Key in case of registered device</li></ul>
Device Info Response | Digital ID | <ul><li>For SBI 1.0 device using device key</li><li>For SBI 2.0 device using FTM chip key</li></ul>
Capture Response | Data | Device key is used
Capture Response | Digital ID | FTM chip key is used
Registration Capture Response | Data | Device key is used
Registration Capture Response | Digital ID | <ul><li>For SBI 1.0 device using device key</li><li>For SBI 2.0 device using FTM chip key</li></ul>
Device Registration Request | Device Data | Device Provider certificate is used
Device Registration Request | Device Info | Device key is used
Device Registration Request | Digital ID | <ul><li>For SBI 1.0 device using device key</li><li>For SBI 2.0 device using FTM chip key</li></ul>
Device De-registration Request | Device | Device Provider certificate is used
Device Registration Response | Response | MOSIP Signature certificate is used
Device Registration Response | Digital ID | Should be same as request
Device De-registration Response | Device | MOSIP Signature certificate is used

## Error Codes
Code    | Message
--------|--------
0       | Success
100     | Device not registered
101     | Unable to detect a biometric object
102     | Technical error during extraction.
103     | Device tamper detected
104     | Unable to connect to management server
105     | Image orientation error
106     | Device not found
107     | Device public key expired
108     | Domain public key missing
109     | Requested number of biometric (Finger/IRIS) not supported
5xx     | Custom errors. The device provider is free to choose his error code and error messages.
