# Setting up MessageMedia SMS for Auth
[![HitCount](http://hits.dwyl.io/messagemedia/auth0-integration.svg)](http://hits.dwyl.io/messagemedia/auth0-integration)

This guide assumes that you've already got your MessageMedia API credentials. If you don't, you can sign up for them for free over [here](https://developers.messagemedia.com/register).

## Create Lambda Function

1. In the [AWS Lambda console](https://console.aws.amazon.com/lambda/home), create a new function and give it a name such as _Auth0SMSGateway_. Use no template.
2. Switch mode from inline code editing to ZIP file upload.
3. Upload `SMSGatewayLambda.zip`.
4. Save your function.

## Create API Gateway

1. In the [AWS API Gateway console](https://console.aws.amazon.com/apigateway/home), create a new gateway and select _Import from Swagger_.
2. Copy the configuration from the file `gateway.swagger.json` and paste it into the Swagger input and create the gateway.
3. Select the created _POST /sendSMS_ resource, make sure to select _Lambda_ as integration type, tick the _Use Lambda Proxy integration_ checkbox and enter the name of your Lambda function (e.g., _Auth0SMSGateway_).
4. Click **Actions**, **Deploy API** and create a stage (e.g., _prod_).
5. Copy the _Invoke URL_ displayed on your screen.

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
    "gateway_url": "{YOUR_GATEWAY_URL}",
    "gateway_authentication": {
        "method": "bearer",
        "subject": "{YOUR_API_KEY}",
        "audience": "{YOUR_API_SECRET}",
        "secret": "MessageMedia"
    },
````

The value for the `gateway_url` is the _Invoke URL_ from the API gateway with the string `/sendSMS` added at the end. It should look something like `https://{ID}.execute-api.{REGION}.amazonaws.com/prod/sendSMS`. Enter your API key and secret as `subject` and `audience` in the `gateway_authentication` section. The `secret` is hardcoded to the string `MessageMedia`.

Send the request (**Try** in the API explorer) to store the configuration.

![Step 5](https://developers.messagemedia.com/wp-content/uploads/2018/09/step5.png)

Once that's done, you can change to the _Try_ tab in the Auth0 UI for the SMS Connection, enter your phone number and hit **Try**. An authentication code message should arrive on your phone.
