# Template: Salesforce new Contact to Twilio SMS
When a new `Contact` is added in Salesforce, send an SMS to a specific number using Twilio.<br>

It is important to be updated with a particular software development tool that you or your team are using in the 
day-to-day development process and get notified immediately on a new release of it. There can be a Slack channel 
followed by a development community or a team that uses that specific tool. This template makes it easier *** each time 
a contact is created in salesforce, Twilio SMS containing all 
the defined fields in contact SObject will be sent.

This template can be used to send a message to a selected Slack channel when a new release is done in a specific 
repository in GitHub.

## Use this template to
- Send a Channel message to a Slack channel which is created for developers who use a Git repository.
- Send a Channel message to a Slack channel which is followed by the contributors of a Git repository.

## What you need
- A Github Account
- A Slack workspace with admin privileges

## How to set up
- Import the template.
- Allow access to the Github account.
- Select the repository.
- Allow access to Slack account.
- Select the channel.
- Set up the template. 

# Developer Guide
<p align="center">
<img src="./docs/images/template_flow.png?raw=true" alt="Github-Slack Integration template overview"/>
</p>

## Supported versions
<table>
  <tr>
   <td>Ballerina Language Version
   </td>
   <td>Swan Lake Alpha2
   </td>
  </tr>
  <tr>
   <td>Java Development Kit (JDK) 
   </td>
   <td>11
   </td>
  </tr>
  <tr>
   <td>Salesforce API 
   </td>
   <td>v48.0
   </td>
  </tr>
  <tr>
   <td>Twilio Basic API
   </td>
   <td>2010-04-01
   </td>
  </tr>
</table>


## Pre-requisites
* Download and install [Ballerina](https://ballerinalang.org/downloads/).
* Twilio account with sms capable phone number
* Ballerina connectors for Salesforce and Twilio which will be automatically downloaded when building the application for the first time


## Configuration
### Setup Salesforce configurations
1. Create a Salesforce account and create a connected app by visiting [Salesforce](https://www.salesforce.com). 
2. Salesforce username, password will be needed for initializing the listener. 
3. Once you obtained all configurations, Replace "" in the `Config.toml` file with your data.

4. Create push topic in Salesforce developer console
  The Salesforce trigger requires topics to be created for each event. We need to configure topic to listen on Custom Object entity.

  * From the Salesforce UI, select developer console. Go to debug > Open Execute Anonymous Window. 
  * Paste following apex code to create topic with <NewContact> and execute. You can change the `pushTopic.Query` adding the fields you want to receive when the event triggered. Find the supported SOQL queries for PushTopic [here](https://developer.salesforce.com/docs/atlas.en-us.api_streaming.meta/api_streaming/supported_soql.htm).
    ```apex
    PushTopic pushTopic = new PushTopic();
    pushTopic.Name = 'NewContact';
    pushTopic.Query = 'SELECT Id, Name, Phone, FirstName, LastName, Email  FROM Contact';
    pushTopic.ApiVersion = 48.0;
    pushTopic.NotifyForOperationCreate = true;
    pushTopic.NotifyForFields = 'Referenced';
    insert pushTopic;
    ```
  * Once the creation is done, specify the topic name in your `Config.toml` file as `sf_push_topic`.

### Setup Twilio configurations
Create a [Twilio developer account](https://www.twilio.com/). 

1. Create a Twilio project with SMS capabilities.
2. Obtain the Account SID and Auth Token from the project dashboard.
3. Obtain the phone number from the project dashboard and set as the value of the `from_mobile` variable in the `Config.toml`.
4. Give a mobile number where the SMS should be send as the value of the `to_mobile` variable in the `Config.toml`.
5. Once you obtained all configurations, Replace "" in the `Config.toml` file with your data.

## Config.toml 
```
[<ORG_NAME>.sdfc_contact_to_twilio]
sf_username = "<SALESFORCE_USERNAME>"  
sf_password = "<SALESFORCE_PASSWORD>"  
sf_push_topic = "<SALESFORCE_PUSHTOPIC>"  
account_sid = "<TWILIO_SID>"  
auth_token = "<TWILIO_AUTH_TOKEN>"  
from_mobile = "<TWILIO_MOBILE_NUMBER>"  
to_mobile = "<TO_MOBILE_NUMBER>"  
```

## Running the template
1. First you need to build the integration template and create the executable binary. Run the following command from the root directory of the integration template. 
`$ bal build`. 

2. Then you can run the integration binary with the following command. 
`$ bal run  bal run target/bin/sdfc_contact_to_twilio-0.1.0.jar`. 

Successful listener startup will print following in the console.
```
>>>>
[2020-09-25 11:10:55.552] Success:[/meta/handshake]
{ext={replay=true, payload.format=true}, minimumVersion=1.0, clientId=1mc1owacqlmod21gwe8arhpxaxxm, supportedConnectionTypes=[Ljava.lang.Object;@21a089fc, channel=/meta/handshake, id=1, version=1.0, successful=true}
<<<<
>>>>
[2020-09-25 11:10:55.629] Success:[/meta/connect]
{clientId=1mc1owacqlmod21gwe8arhpxaxxm, advice={reconnect=retry, interval=0, timeout=110000}, channel=/meta/connect, id=2, successful=true}
<<<<
```

3. Now you can create a new contact in Salesforce Account and observe that integration template runtime has received the event notification for the created contact.

4. You can check the SMS received to verify that information about the contact creation is received. 
