# Mobile Authentication Framework for Android developers

## Overview
This project is an open-source code that provides an example of how to use [PingIdentity Authentication API](https://docs.pingidentity.com/bundle/pingfederate-93/page/qsl1564002999029.html)  *browserless* OAuth flow in Android projects. It supports selected MFA flows and provides the current state of a flow, as an end-user steps through a PingFederate authentication policy. This native Android code allows end-user interactions with the API, such as credential prompts, to be handled by an external mobile application.
The repository contains two modules `PingAuthenticationCore` (*Core*) and `PingAuthenticationUI` (*UI*) where *Core* is an independent module and *UI* depends on it.

*Note: those modules are designed to work in conjunction with PingID SDK or PingOne SDK and will not work separately.*

### PingAuthenticationCore
This is the main module that is responsible for communication with the OpenID Connect issuer. It manages flow states and contains all the needed configurations in one place, making it easy to configure and work with. This module is independent, in that it can work without the `PingAuthenticationUI` module, which is actually an example of how to use public APIs of the *Core*.

### PingAuthenticationUI
This is an example module that demonstrates *one of the possible ways* to implement a user interface that will handle all the various states from `PingAuthenticationCore`. This is the reason that this module depends on `PingAuthenticationCore`.

Release notes can be found [here](./release-notes.md).

### Documentation

Reference documentation about PingFederate is available here: 
* [Introduction to PingFederate](https://docs.pingidentity.com/csh?Product=pf-latest&topicname=tle1564002955874.html)

### Prerequisites

1. In order to use the Mobile Authentication Framework need to set an app with Ping Identity SDK. For a full sample code of the Mobile Authentication Framework integrated with Sample app, please check [PingOne SDK Sample App](https://github.com/pingidentity/pingone-mobile-sdk-android).

2. Prepare the PingFederate OAuth client (application):
* [Authentication API](https://docs.pingidentity.com/bundle/pingfederate-93/page/qsl1564002999029.html)
* [Installing PingFederate](https://docs.pingidentity.com/csh?Product=pf-latest&topicname=ptg1564002959252.html)
* [Managing OAuth clients](https://docs.pingidentity.com/csh?Product=pf-latest&topicname=hsx1564002992533.html)
* [Integration with the PingFederate Authentication API for PingID SDK](https://apidocs.pingidentity.com/pingid-sdk/guide/pf-adapter/pid_c_SDKadapterForPFauthenticationApi/)
* [Integration with the PingFederate Authentication API for PingOne MFA](https://docs.pingidentity.com/bundle/integrations/page/vyz1605884824044.html)

Note: The Mobile Authentication Framework for Android supports different versions per platform:
* PingID: PingFederate 10.1, Adapter 1.8 and up.
* PingOne: PingFederate 9.3, Adapter 1.0 and up.

## Getting started

1. Clone this repository and import *both* folders `PingAuthenticationCore` and `PingAuthenticationUI` as **separate modules** to your existing project that uses PingID SDK or PingOne SDK library. (File -> New... -> Import module... in Android Studio)
2. Put your desired configuration in the `gradle.properties` file of the `PingAuthenticationCore` module:
* `OIDC_ISSUER`: The base URL for the PingFederate endpoint. 
* `CLIENT_ID`: The ID of the client as configured in the PingFederate console.
* `RESPONSE_TYPE`: Valid values are "token" or "code". If it is "code", an extra API call generates the accessToken.
* `SCOPE`: By default it is set to "openid", and can be modified to support the adapter scope.
* `RESPONSE_MODE`: By default it is set to "pi.flow", and can be modified to support different adapter response mode.
* `CLIENT_SECRET`: Optional value, according to its configuration in PingFederate.
* `GRANT_TYPE`:  By default it is set to “authorization_code”, for the token exchange code request.
3. If you are using `PingAuthenticationUI` module add to your `AndroidManifest.xml` the following line:
```java
 <activity android:name="com.pingidentity.authenticationui.PingAuthenticationUIActivity"/>
 ```
4. `PingAuthenticationUI` module uses [Android Navigation Pattern](https://developer.android.com/guide/navigation/navigation-principles), thus if you are using this module as-is add in the **project-level** `build.gradle` file the following line under dependencies (if not exists):
```java
classpath "androidx.navigation:navigation-safe-args-gradle-plugin:2.3.1"
``` 

*For full end-to-end example of one of the possible ways of using the Mobile Authentication Framework for Android please check our [PingOne SDK Sample Application](https://github.com/pingidentity/pingone-mobile-sdk-android/tree/master/Sample%20Code).*

## Public API's

### PingAuthenticationUI
```java
/**
 * Starts an Authentication process with PingIdentity Authentication API. Calling this method will open a new Activity
 * in a context of provided one. The provided activity will also get the result of authentication process.
 * @param context - an Activity that will retrieve a result of authentication process.
 * @param mobilePayload - a String retrieved from PingID SDK or PingOne SDK
 * @param dynamicData - a String for passing any additional data passed from the app to the Authentication API. 
 */
public void authenticate(@NonNull Activity context, @NonNull String mobilePayload, @Nullable String dynamicData)
```

```java
/**
 * The authentication flow with Mobile Authentication framework may require from the end-user to pair current mobile device.
 * In this case it will return to hosting Activity with a serverPayload. After pairing your device you need to call this
 * method to continue authentication process.
 * @param context - an Activity wich will retrieve the result of authentication process.
 */
public void continueAuthentication(Activity context)
```

### PingAuthenticationCore
**Please note, that if you are using PingAuthenticationUI module as-is you would never need to call this API directly**
```java
/** 
 * Executes a request according to the RequestParams and delivers response through AuthenticationCallback
 * @param params - RequestParams object, can be null at initial request
 * @param callback - AuthenticationCallback that will be triggered on AuthenticationState change
 */
public static void authenticate(@Nullable RequestParams params, @NonNull AuthenticationCallback callback)
```

## Retrieving results
The results of Mobile Authentication Framework process will be returned in the method `onActivityResult` of hosting Activity of your application. 
```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //checking if the result received from Mobile Authentication Framework
        if(requestCode == PingAuthenticationUI.AUTHENTICATION_UI_ACTIVITY_REQUEST_CODE){
            if (resultCode == Activity.RESULT_OK){
                if(data!=null && data.hasExtra("serverPayload")){
                    /*
                     * Received server payload from Mobile Authentication Framework. This means you need to pair your device
                     * and then call continueAuthentication(Activity context) method to continue.
                     */
                }
                if (data!=null && data.hasExtra("access_token")){
                    /*
                     * The access token has been succesfully received. Flow ended.
                     */
                    
                }
            }else{
                // Mobile Authentication Flow was canceled. Do nothing
            }
        }else{
            //another request code can be handled in the same method
        }
    }
```

## Disclaimer

THE SAMPLE CODE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SAMPLE CODE OR THE USE OR OTHER DEALINGS IN
THE SAMPLE CODE.  FURTHERMORE, THIS SAMPLE CODE IS NOT COMMERCIALLY SUPPORTED BY PING IDENTITY BUT QUESTIONS MAY BE ADDRESSED TO PING'S SUPPORT CENTER OR MAY BE OTHERWISE ADDRESSED IN THE RELATED DOCUMENTATION.

Any questions or issues should go to the support center, or may be discussed in the [Ping Identity developer communities](https://support.pingidentity.com/s/topic/0TO1W000000atTxWAI/pingone-mfa).
