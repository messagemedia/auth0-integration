# Setting up MessageMedia SMS for Auth
[![HitCount](http://hits.dwyl.io/messagemedia/auth0-integration.svg)](http://hits.dwyl.io/messagemedia/auth0-integration)

This guide assumes that you've already got your MessageMedia API credentials. If you don't, you can sign up for them for free over [here](https://developers.messagemedia.com/register).

## Auth0 Configuration

Make sure the SMS Passwordless connection is enabled by going to [Connections → Passwordless in your Auth0 account](https://manage.auth0.com/#/connections/passwordless) and switch it on. You can configure everything through the UI except for choosing a different provider. You can enter dummy values for Twilio SID and AuthToken.

To change the provider to MessageMedia, you need to use the Management API. To get an access token, go to the [API Explorer tab of your Auth0 Management API](https://manage.auth0.com/#/apis/management/explorer). You can use the API Explorer directly, curl or Postman.

![Step 1](https://developers.messagemedia.com/wp-content/uploads/2018/09/step1.png)

If you use the API Explorer, you need to add the access token and tenant URL to it:

![Step 2](https://developers.messagemedia.com/wp-content/uploads/2018/09/step2.png)

Using your access token, make a GET request to `https://{YOURTENANT}.auth0.com/api/v2/connections` to find out the ID for your SMS connection. You can make that request [in the API Explorer](https://auth0.com/docs/api/management/v2#!/Connections/get_connections) by clicking **Try**.

![Step 3](https://developers.messagemedia.com/wp-content/uploads/2018/09/step3.png)

Go through the returned JSON to find the connection named `sms` and take note of the ID to use in the next step. Also, copy the `options` object.

![Step 4](https://developers.messagemedia.com/wp-content/uploads/2018/09/step4.png)

Next, make a PATCH request to `https://{YOURTENANT}.auth0.com/api/v2/connections/{CONNECTION_ID}`, using the connection ID you found. You can do that [in the API Explorer](https://auth0.com/docs/api/management/v2#!/Connections/patch_connections_by_id). The body of the request must be the existing `options` object with these edits: remove `twilio_sid` and `twilio_token` and instead add the following entries.

````json
    "provider": "sms_gateway",
    "gateway_url": "https://auth0.workflows-prd.syd.mmd.zone/messages",
    "gateway_authentication": {
        "method": "bearer",
        "subject": "{YOUR_API_KEY}",
        "audience": "{YOUR_API_SECRET}",
        "secret": "MessageMedia"
    },
    "from": "{SENDER_PHONE_NUMBER}",
````

Enter your Message Media API key and secret as `subject` and `audience` in the `gateway_authentication` section. The `secret` is hardcoded to the string `MessageMedia`. `from` is the sender's phone number.

Send the request (**Try** in the API explorer) to store the configuration.

![Step 5](https://developers.messagemedia.com/wp-content/uploads/2018/09/step5.png)

Once that's done, you can change to the _Try_ tab in the Auth0 UI for the SMS Connection, enter your phone number and hit **Try**. An authentication code message should arrive on your phone.
