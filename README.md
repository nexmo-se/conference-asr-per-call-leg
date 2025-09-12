# Conference - Separate ASR for each call leg of a multi-party conference

This is a sample application using Vonage Voice API to do separate ASR (Automatic Speech Recoginition) for each call leg of a multi-party conference call.

You may use this Voice API application to connect voice calls to an ASR engine using the Connector listed in the Set up section.

Voice calls attached to the conference may be:</br>
inbound/outbound,</br>
PSTN calls (cell phones, landline phones, fixed phones),</br>
SIP calls with [SIP endpoints](https://developer.vonage.com/en/voice/voice-api/concepts/endpoints#session-initiation-protocol-sip) or [Programmable SIP](https://developer.vonage.com/en/voice/voice-api/concepts/programmable-sip),</br>
[WebRTC](https://developer.vonage.com/en/vonage-client-sdk/overview) calls (iOS/Android/Web Javascript clients).</br>

## About this sample Voice API application

This application connects voice calls to the ASR Connector server by using the [WebSockets feature](https://developer.vonage.com/en/voice/voice-api/concepts/websockets) of Vonage Voice API.</br>

When a new voice call is established and attached to a conference, this Voice API application triggers a WebSocket connection from Vonage platform to the ASR Connector server which streams audio from the participant's call leg to the ASR engine. 

Instead of using this sample Voice API application, you may use your own existing Voice API application to establish WebSockets with the ASR Connector server to connect your managed voice calls with the ASR engine.

Your new or existing Voice API application may be written with any programming language using [server SDKs](https://developer.vonage.com/en/tools) or with direct [REST API](https://developer.vonage.com/en/api/voice) calls.

You may also have your Vonage [Video API WebRTC Clients](https://developer.vonage.com/en/tools) establish sessions with the ASR engine using the Video API's [Audio Connector](https://developer.vonage.com/en/video/guides/audio-connector) and the peer Connector server as listed in the next section, in which case this Voice API application is not needed.

## Set up

### Set up the ASR Connector server - Host server public hostname and port

First set up the ASR Connector server from the repository</br>
https://github.com/nexmo-se/deepgram-connector</br>

Default local (not public!) of the Connector server `port` is: 6000.

If you plan to test using a `Local deployment`, you may use ngrok (an Internet tunneling service) for both<br>
this Voice API application<br>
and the Connector application<br>
with [multiple ngrok tunnels](https://ngrok.com/docs/agent/config/v2/#tunnel-configurations).

To do that, [install ngrok](https://ngrok.com/downloads).<br>
Log in or sign up with [ngrok](https://ngrok.com/),<br>
from the ngrok web UI menu, follow the **Setup and Installation** guide.

Set up two tunnels,<br>
one to forward to the local port 6000 (as the Connector application will be listening on port 6000),<br>
the other one to the local port 8000 for this Voice API application,<br>
see this [sample yaml configuration file](https://ngrok.com/docs/agent/config/v2/#define-two-tunnels-named-httpbin-and-demo), but it needs port 6000 and 8000 as actual values,<br>
depending if you have a paid ngrok account or not, you may or may not be able to set (static) domain names.

Start ngrok to start both tunnels that forward to local ports 6000 and 8000, e.g.<br>
`ngrok start httpbin demo`

please take note of the ngrok Enpoint URL that forwards to local port 6000 as it will be needed here for this Voice API application environment variable as **`PROCESSOR_SERVER`** in one of the next sections, that URL looks like:<br>
`xxxxxxxx.ngrok.xxx` (for ngrok),<br>
or `myserver.mycompany.com:32000` (public host name and port of your Connector application server)<br>
no `port` is necessary with ngrok as public host name,<br>
that host name to specify must not have leading protocol text such as `https://`, `wss://`, nor trailing `/`.

### Set up your Vonage Voice API application credentials and phone number

[Log in to your](https://dashboard.nexmo.com/sign-in) or [sign up for a](https://ui.idp.vonage.com/ui/auth/registration) Vonage APIs account.

Go to [Your applications](https://dashboard.nexmo.com/applications), access an existing application or [+ Create a new application](https://dashboard.nexmo.com/applications/new).

Under Capabilities section (click on [Edit] if you do not see this section):

**Enable** Voice
- Under Answer URL, leave HTTP GET, and enter</br>
https://\<host\>:\<port\>/answer</br>
(replace \<host\> and \<port\> with the public host name and if necessary public port of the server where this sample application is running)</br>
- Under Event URL, **select** HTTP POST, and enter</br>
https://\<host\>:\<port\>/event</br>
(replace \<host\> and \<port\> with the public host name and if necessary public port of the server where this sample application is running)</br>
Note: If you are using ngrok for this sample application, the answer URL and event URL look like:</br>
https://yyyyyyyy.ngrok.xxx/answer</br>
https://yyyyyyyy.ngrok.xxx/event</br> 	
- Click on [Generate public and private key] if you did not yet create or want new ones, save the private key file in this application folder as .private.key (leading dot in the file name).</br>
- Click on [Generate new application] if you've just created the application.</br></br>

**IMPORTANT**: If you already had an existing application and just created a new key set, do not forget to click on [Save changes] at the bottom of the screen.</br></br>

- Link a phone number to this application if none has been linked to the application.</br>

Please take note of your **application ID** and the **linked phone number** (as they are needed in the very next section).

For the next steps, you will need:</br>
- Your [Vonage API key](https://dashboard.nexmo.com/settings) (as **`API_KEY`**)</br>
- Your [Vonage API secret](https://dashboard.nexmo.com/settings), not signature secret, (as **`API_SECRET`**)</br>
- Your `application ID` (as **`APP_ID`**),</br>
- The **`phone number linked`** to your application (as **`SERVICE_PHONE_NUMBER`**), your phone will **call that number**.</br>

### Local deployment

Copy or rename .env-example to .env<br>
Update parameters in .env file<br>
Have Node.js installed on your system, this application has been tested with Node.js version 22.16.0<br>

Install node modules with the command:<br>
 ```bash
npm install
```

Launch the application:<br>
```bash
node conference-asr-per-call-leg
```
Default local (not public!) of this application server `port` is: 8000.

### How to make PSTN calls

#### Inbound calling

Call the **`phone number linked`** to your application to get connected to a Conference, the caller will be asked to enter a Conference number.

#### Outbound calling

To manually trigger an outbound PSTN call to a number (that will be connected to a Conference), open a web browser, enter the address:<br>

_https://\<server-address\>/call?callee=\<number\>_<br>

the \<number\> must be in E.164 format without leading '+' sign, or '-', '.' characters

for example, it looks like

https://xxxx.ngrok.app/call?callee=12995551212

Upon answering the call, the callee will be asked to enter a Conference number.

Of course, you may programmatically initiate outbound calls by using the API call listed in the corresponding webhook section of the program _conference-asr-per-call-leg.js_ (i.e. `/call`).

## Additional resources

If you have questions, join our [Community Slack](https://developer.vonage.com/community/slack) or message us on [X](https://twitter.com/VonageDev?adobe_mc=MCMID%3D61117212728348884173699984659581708157%7CMCORGID%3DA8833BC75245AF9E0A490D4D%2540AdobeOrg%7CTS%3D1740259490).



