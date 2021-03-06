# Table of Contents
- [Custom Notifications Smart Connector](#custom-notifications-smart-connector)
    + [XML SDK documentation](#xml-sdk-documentation)
    + [Usage](#usage)
      - [Connector Installation (Publishing into Exchange)](#connector-installation--publishing-into-exchange-)
      - [Anypoint Runtime Manager configuration (Alerts)](#anypoint-runtime-manager-configuration--alerts-)      
      - [Connector configuration (Mule Application)](#connector-configuration--mule-application-)
        * [Examples](#examples)
          + [Sending INFO notifications](#sending-info-notifications)
          + [Sending ERROR notifications](#sending-error-notifications)
    + [Exceptions](#exceptions)
    + [Change the HTML template](#change-the-html-template)
    + [Considerations and limitations](#considerations-and-limitations)
    + [Appendix I - Anypoint Exchange documentation](#appendix-i---anypoint-exchange-documentation)
    + [Release Notes](#release-notes)
        * [1.0.0-SNAPSHOT:](#100-snapshot-)
    + [Collaborations](#collaborations)

# Custom Notifications Smart Connector
This is a smart connector built using XML SDK (Mule 4) and it's only applicable for Cloudhub based deployment models. The goal of this custom connector is to simplify the way an application can send notifications. This custom connector proposes a strategy to send personalised notifications using the platform's proposed out of the box components (Cloudhub Connector and Runtime Manager Alerts). Although the Cloudhub Connector itself doesn't represent too much complexity, this custom connector speeds up the practice and achieves better reusability, mainly by removing the need of each application that must send a notification has to know the HTML template used, parse it and configure the notification on top.

The notifications sent by this custom connector override the notification model used out of the box by the Platfom, replacing it with a simple form. All this performed in an Async fashion, to reduce the overhead introduced by the notification strategy.
  
### XML SDK documentation
https://docs.mulesoft.com/mule-sdk/1.1/xml-sdk

### Usage
The usage of this connector involves different stages, listed below:
* connector installation (publishing into Exchange) - 1 time only task
* Platform configuration for Alerts - 1 time only task
* Mule application usage

#### Connector Installation (Publishing into Exchange)
Execute the following steps on the folder where the project was cloned.
1. Make sure to replace the <groupId> tag on the pom.xml with your organization ID.
2. Execute the following in a terminal/cmd/shell:
```mvn deploy -DskipTests```
The output of this command would be something like:
```
[INFO] --- maven-deploy-plugin:2.8.2:deploy (default-deploy) @ custom-notification-extension ---
[INFO] No primary artifact to deploy, deploying attached artifacts instead.
...
Uploading to : https://maven.anypoint.mulesoft.com/api/v2/organizations/8033ff23-884a-4352-b75b-14fc8237b2c4/maven/8033ff23-884a-4352-b75b-14fc8237b2c4/custom-notification-extension/1.0.0-SNAPSHOT/maven-metadata.xml
Uploaded to : https://maven.anypoint.mulesoft.com/api/v2/organizations/8033ff23-884a-4352-b75b-14fc8237b2c4/maven/8033ff23-884a-4352-b75b-14fc8237b2c4/custom-notification-extension/1.0.0-SNAPSHOT/maven-metadata.xml (1.1 kB at 1.1 kB/s)
Uploading to : https://maven.anypoint.mulesoft.com/api/v2/organizations/8033ff23-884a-4352-b75b-14fc8237b2c4/maven/8033ff23-884a-4352-b75b-14fc8237b2c4/custom-notification-extension/maven-metadata.xml
Uploaded to : https://maven.anypoint.mulesoft.com/api/v2/organizations/8033ff23-884a-4352-b75b-14fc8237b2c4/maven/8033ff23-884a-4352-b75b-14fc8237b2c4/custom-notification-extension/maven-metadata.xml (361 B at 361 B/s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  31.408 s
[INFO] Finished at: 2020-08-25T14:13:28-03:00
[INFO] ------------------------------------------------------------------------
```
    
#### Anypoint Runtime Manager configuration (Alerts)
In order to trigger the notifications (email) we need to configure alerts in the Runtime Manager. 
Reference: https://docs.mulesoft.com/runtime-manager/alerts-on-runtime-manager
It's recommended to configure to different alerts, one Warning/Critical for errors and one Info for info notifications. To distinguish between both is important to set the "Condition" of the alert to "Custom application notification" and set the proper Priority as follows:
| Priority       | Contains  |
| ------------- |:-------------:|
|Error |ERROR|
|Info or Any|INFO|

The value on the **Contains** column will be matched with the values set by the connector for the ***Type*** field of the HTML template. This value will be inferred based on the values sent.

Here is an example of both alerts configured:
![image info](./images/info-alert.png)
![image info](./images/error-alert.png)


#### Connector configuration (Mule Application)
The use of the connector during the development phase is very straigthforward:
1. Get the connector from Exchange. Reference: https://docs.mulesoft.com/studio/7.6/add-modules-in-studio-to
2. Use it as indicated on the examples
##### Connector Configuration
Next you'll find examples to send INFO or ERROR notifications. This explanation assumes that Anypoint Platform username and password were set creating a new configuration and that configuration was associated with the connector, as follows:

![image info](./images/platform-credentials.png)

###### Sending INFO notifications
This option is useful when we want to notify about certain business events that are not errors, but are important for decisioning (i.e. execution results, metrics of a batch/poller process, etc). Please take into account that including notifications steps adds some overhead to the final process (less than 2%) and some of this messages can be obtained performing a proper analysis of the logs files. It's advisable not to abuse of these types of notifications.

The following sample flow will be used to demonstrate this scenario
![image info](./images/example-info-app.png)

This is just a simple HTTP listener listening on default port. When an incoming request is received, then the connector is executed, sending a notification via email.
The following connector's properties are mandatory to send INFO notifications:

| Property       | description  |
| ------------- |:-------------:|
|Application Name|{string\|dw expression} allowed|
|Message|{string\|dw expression} allowed. Tab "Notification Details". Objects are accepted|

![image info](./images/info-config.png)

As a result, the following email notification is triggered:
![image info](./images/info-email.png)

###### Sending ERROR notifications
This option is useful when we want to notify about errors that happened during the execution of Mule Apps. The errors should be also stored in log files and this connector is not a replacement for these.

The following sample flow will be used to demonstrate this scenario
![image info](./images/example-error-app.png)

This is just a simple HTTP listener listening on default port. When an incoming request is received, then an HTTP request is perform to an endpoint that will reply with a timeout error. When the exception is catched, the connector is executed, sending a notification via email.

The following connector's properties are mandatory to send ERROR notifications:

| Property       | description  |
| ------------- |:-------------:|
|Application Name|{string\|dw expression} allowed|
|Error|Error object autogenerated by Mule. This can be captured using the following dw expression:  ```#[error]```. Tab "Notification Details".|

![image info](./images/error-config.png)

As a result, the following email notification is triggered:
![image info](./images/error-email.png)

### Exceptions
* ***MODULE-CUSTOMNOTIFICATION:EMPTY_MESSAGE***. This error is thrown when none, the Error and Message properties is sent. Both are OPTIONALS by nature on its definition, but one of these should be present.
* ***MODULE-CUSTOMNOTIFICATION:DUAL_MESSAGE***. This error is thrown when both, the Error and Message properties are sent. Both are OPTIONALS by nature on its definition, but only one should be present.

### Change the HTML template
The template used is embedded in the logic of the connector (module-CustomNotification.xml), as an escaped single line HTML. To modify it, it's recommended to first make an unescape and then format it. Otherwise it is very difficult to manipulate.
There are online tools that make this procedure very easy.

### Considerations and limitations
* This connector is using the Cloudhub connector (https://docs.mulesoft.com/cloudhub-connector/1.0/), therefore the same limitations applies to this connector as well. 
* The retention limitation of Runtime Manager for Notifications, explained in https://docs.mulesoft.com/runtime-manager/notifications-on-runtime-manager#notification-retention
* The rate limit of Runtime Manager for Alerts, explained in https://docs.mulesoft.com/runtime-manager/alerts-on-runtime-manager#rate-limits-on-alerts
* The size of the HTML template used. Make sure to not exceed the 10MB for the HTML page generated after parsing the template. The lighter the better.
* This connector only works with non-federated users or connected apps. 
* This isn't an official connector provided by MuleSoft. This means that if there is any issue in an integration that is using this connector, the support is out of scope of the MuleSoft Support team. If you open a ticket, this team will probably request a simplified version of the application (removing all the custom components that make it up)
* This connector was made using the MuleSoft XML SDK, so a set of limitations applies on the connector. Take this into account for future enhancement you may want to add. Reference: https://docs.mulesoft.com/mule-sdk/1.1/xml-sdk#xml-sdk-limitations

### Appendix I - Anypoint Exchange documentation
When this connector is deployed to Exchange, this generates an asset. Then, Exchange automaticaly creates a landing page that will serve as a reference for anyone who wants to consume it. It's a good practice to provide substantial documentation regarding its use. This documentation can follow a format similar to the steps explained here, in this README page. please, before doing so make the pertinent changes, especially the onees required to the use of images.

### Release Notes
   - ##### 1.0.0-SNAPSHOT: 
      - **Description**: Initial SNAPSHOT version
      - **Todo list**:
        | category       | description  |
        | ------------- |:-------------:|
        | Munit         | Add MUnit to connector|

### Collaborations
Want to collaborate? awesome! we're using a simplified gitflow workflow. Clone your repo locally, create a feature branch using the naming convention ```feature/name-of-the-feature``` and, once its ready, push your changes and open a pull request.
