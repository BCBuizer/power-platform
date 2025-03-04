---
title: Specify which emails are automatically tracked
description: Use email message filtering and correlation to specify which emails are tracked. 
author: sericks007
ms.component: pa-admin
ms.topic: conceptual
ms.date: 03/10/2023
ms.subservice: admin
ms.author: sericks
ms.contributor: dmartens
search.audienceType: 
  - admin
search.app:
  - D365CE
  - PowerApps
  - Powerplatform
  - Flow
---
# Specify which emails are automatically tracked

With server-side synchronization and Dynamics 365 App for Outlook, you can automatically create email activities in customer engagement apps (such as [Dynamics 365 Sales](/dynamics365/sales-professional/help-hub), [Dynamics 365 Customer Service](/dynamics365/customer-service/help-hub), [Dynamics 365 Marketing](/dynamics365/marketing/help-hub), [Dynamics 365 Field Service](/dynamics365/field-service/overview), and [Dynamics 365 Project Service Automation](/dynamics365/project-operations/psa/overview)). These apps are based on received email messages. 

> [!IMPORTANT]
> Only emails directly within the Inbox folder of a mailbox are evaluated for automatic tracking. If an email is in a subfolder, the item would only be processed if the folder is configured for [folder-level tracking](../admin/configure-outlook-exchange-folder-level-tracking.md). If an email message was in a subfolder when Server-Side Synchronization processed the mailbox, moving the email back into the Inbox folder will not result in the email being processed during the next sync cycle. Server-Side Sync tracks the received date of the last email that was processed so during the next sync cycle it can evaluate only the emails received after that date/time. To process an email that was not within the Inbox folder, you can use one of the following options:
> - Manually track the email using [Outlook category tracking](use-outlook-category-track-appointments-emails.md) or [Dynamics 365 App for Outlook](/power-apps/user/use-outlook-app)
> - Update the Process Emails From date on the mailbox record to a date prior to when the email was received. This field is not exposed on the Mailbox record form by default.
> - Click the [Test & Enable Mailbox](connect-exchange-online.md#test-the-configuration-of-mailboxes) button within the mailbox record in Dynamics 365. 

With automation to track email messages users can select a filter option that determines which email messages are tracked. 

Users can set the following email filter options in the **Set Personal Options** dialog box on the **Email** tab. For more information, see [Set personal options that affect tracking and synchronization between customer engagement apps and Outlook or Exchange](set-personal-options-affect-tracking-synchronization-between-dynamics-365-outlook-exchange.md).


- **All email messages**: All email messages received by the user are tracked in Dynamics 365 as an email activity.

- **Email messages in response to email**: Only replies to email messages that have already been tracked are saved as email activities in Dynamics 365.

- **Email messages from Leads, Contacts, and Accounts**: Only email messages sent which resolve to a lead, contact, or account row in Dynamics 365 is saved as activities.

- **Email messages from records that are email enabled**: Email messages are tracked from any row  that resolves to an email address in Dynamics 365, including custom tables.

- **No email messages**: No email messages received by the user are tracked (will have activities created). This option only affects auto tracked emails.

> [!NOTE]
> These options are configurable to automatically track sent items and can also be configured for queues and queue mailboxes.

   > [!div class="mx-imgBorder"] 
   > ![Convert incoming eamils.](media/email-filter-image1.png)

Emails that are manually tracked by a user in Outlook via [Dynamics 365 App for Outlook](/dynamics365/outlook-app/dynamics-365-app-outlook-user-s-guide), [folder-level tracking](configure-outlook-exchange-folder-level-tracking.md), or [Outlook category](use-outlook-category-track-appointments-emails.md) are synchronized with Dynamics 365 regardless of the configured filter options, as the user wants to manually track the email into Dynamics 365.

The **Email messages sent in response to email** options is turned on by default. Correlation occurs after an email message is filtered. An administrator can turn off all message tracking for a particular user by setting **Incoming Email** under **Synchronization Method** to **None** on the **Mailbox** form. This means server-side synchronization won't process the mailbox for incoming emails, so no incoming emails from the mailbox will be tracked in Dynamics 365.

An administrator can by enable or disable email correlation from the **[System Settings](system-settings-dialog.md)** dialog box on the **Email** tab at the organization level.

## Use conversations to track emails  

Dynamics 365 uses the following information from an email to determine if a new email correlates to an existing email message in Dynamics 365:


- **MessageId** (hidden in email client user interface but stamped on the email message header): Uniquely identifies an email message which is stamped on every email message.

- **InReplyTo** (hidden in email client user interface but stamped on the email message header): Contains the **messageId** that the email message is in reply to.

   > [!NOTE]
   > Server-side synchronization uses the In-Reply-To header of an email to identify an email that is a reply to another email. However, Dynamics 365 isn't able to control the behavior of different email clients that may be used to reply to an email. For example, if you reply to an email and change the subject, some email clients may remove the In-Reply-To value from the message headers. This prevents Dynamics 365 from being able to correlate the email to a previous email based on In-Reply-To. A [recent update](/officeupdates/semi-annual-enterprise-channel-preview#outlook-4) to the desktop version of Microsoft Outlook changed behavior so the In-Reply-To header isn't removed when replying to an email and the subject was changed.

- **ConversationIndex** (hidden in email client user interface but stamped on the email message header): Contains data that associates an email message to an email thread.

- **Tracking token in subject** (visible in the email subject): Dynamics 365 concept that's stamped directly in the subject line.

- **Subject words and recipients**: Smart matching is a Dynamics 365 concept that uses this information based on configuration.


The email correlation logic goes through each of theses correlation options, in this order:


1. **In reply to correlation**: When an email message is analyzed for correlation, if the **inreplyto** header is present and contains a **messageId** of an email message that's already in Dynamics 365, the email is correlated to existing email message. This is set to **ON** by default, which is recommended. The email correlation is accurate, there is no false positives.

    > [!div class="mx-imgBorder"] 
    > ![Use correlation to track eamil converstions.](media/email-filter-image2.png)


2. **Tracking token correlation**: When an email message is analyzed for correlation, if the subject contains a tracking token, then the system searches for existing emails in Dynamics 365 looking the same tracking token. If there's a match, then the new email message is tracked and related to it the existing email. This option is set to **On** by default.

   > [!NOTE]
   > Tracking token information is visible to the user and it can be changed which can cause an email to correlate with the wrong email.

     > [!div class="mx-imgBorder"] 
     > ![Use tracking token.](media/email-filter-image3.png)

3. **Conversation Index:** When an email message is analyzed for correlation, if a conversationIndex is present in the headers, and another email message is found in dynamics with the same conversationIndex header, the email message will be correlated


    > [!div class="mx-imgBorder"] 
    > ![User correlation to track email converstions.](media/email-filter-image4.png)

4. **Smart matching**: When an email message is analyzed for correlation, the recipients and the words in the subject are extracted, email messages with similar recipients and enough matching subject words will be correlated to it. For more information, see [Smart matching](email-message-filtering-correlation.md#smart-matching).

   > [!NOTE]
   > Smart matching can cause email correlation to hit false positives, hence it is not recommended to enable it. However, it is available in case you need to correlate emails which don't belong to the same email thread. 


    > [!div class="mx-imgBorder"] 
    > ![User smart matching.](media/email-filter-image5.png)


<a name="BKMK_tracking-token"></a>   

## How to determine if and why an email was automatically tracked
As mentioned above in [Use conversations to track emails](email-message-filtering-correlation.md#use-conversations-to-track-emails), emails may be automatically tracked based on the settings configured for your organization and the settings of individual users or queues. There are multiple columns in the email table which can be used to identify if and why an email was automatically tracked. These columns can be added to an Advanced Find view of emails to help understand why an email was tracked and if it was correlated with a previously tracked email.

|                  Column                   |            Description           |
|-----------------------------------------|------------------------------------|
|                 Accepting Entity             |    The user or queue that received the email and was configured to automatically track it. For instance, if an email is received by a queue named Sales (`sales@contoso.com`) that has been configured to track all emails, then the Sales queue would be considered the Accepting Entity. |
|                 Correlated Activity ID       |  Indicates whether an email was associated to a previously tracked email. For example, if an email was sent from Dynamics 365 and a subsequent reply was automatically tracked, the Correlated Activity ID of the reply would reference the original email that was sent.
| Correlation Method | The correlation method used to automatically track an email. It's not currently available for Advanced Find or other views and forms, but you can use the Web API to view it. To see the correlation method for a specific email, use this URL format: <br><br>`https://YourDynamics365URL/api/data/v9.2/emails(IDofEmail)?$select=subject,correlationmethod` <br><br>For example, if your Dynamics 365 URL is `https://contoso.crm.dynamics.com`, you can use this URL to view the correlation method used for an email with an ID of fd372987-7fac-ed11-aad1-0022480819b5: <br><br>`https://contoso.crm.dynamics.com/api/data/v9.2/emails(fd372987-7fac-ed11-aad1-0022480819b5)?$select=subject,correlationmethod` <br><br>To find the ID of an email, open the email and check the URL. <br><br>In the above example, the URL for the email would end with **&id=fd372987-7fac-ed11-aad1-0022480819b5**. The value you see for correlation method will be a number, which corresponds to a specific correlation method. Refer to the table in the correlation method section of the [email EntityType](/power-apps/developer/data-platform/webapi/reference/email?view=dataverse-latest#properties&preserve-view=true) to understand which correlation method is represented by the number. For example, a value of 3 indicates that the email was correlated based on the InReplyTo method.
|     Parent Activity ID   |     This column is used to reference a previously tracked email if the current email is correlated with it. However, the column will only be populated if the email was correlated using the InReplyTo or ConversationIndex correlation method. <br><br>For instance, if an email was sent from Dynamics 365 and a reply to that email was automatically tracked based on the InReplyTo or ConversationIndex methods, the Parent Activity ID column of the email reply would reference the sent email. In cases where the email was correlated using a different method, such as Tracking Token, the Parent Activity ID column would be empty.   |
|         Receiving Mailbox     |      This column displays the mailbox that was processed by server-side synchronization when an email was detected to contain a user or queue that was configured to automatically track it. If the [OrgDBOrgSetting](../admin/OrgDbOrgSettings.md) called SSSForceFilteringMethodForUserMailboxes is disabled (which is the default setting), the value of this column may differ from the Accepting Entity. <br><br>For instance, suppose an email is sent to a user named Paul Cannon, who is configured to automatically track only replies to existing emails. The same email is also received by a queue named Sales (`sales@contoso.com`) that is set up to track all emails. Server-side synchronization may process Paul's mailbox first and recognize that there is a recipient of the email (Sales) that is configured to automatically track the email. In this case, the Accepting Entity would be the Sales queue, but the Receiving Mailbox would be Paul Cannon's mailbox.  |

## Automatic population of Regarding column
The Regarding column is present in each email row and is used to link the email to another row. For instance, the email might be linked to an opportunity, order, or case. If you are manually creating or updating an email row, you have the option to use the Regarding column to connect the email to a row from any table that has been set up to permit activities. If the email is tracked automatically because it was correlated with a previously tracked email, the Regarding column is automatically assigned the same value as the already tracked email.

## How customer engagement apps use tracking tokens  
A tracking token is an alphanumeric string generated by customer engagement apps and appended to the end of an email subject line. It matches email activities with email messages.

Tracking tokens add an additional correlation component. When your app generates an outgoing email activity, a resulting email response arriving in the Dynamics 365 apps system is then correlated to the originating activity.

By default, the tracking token feature is turned on.

### Tracking token structure  

By default, customer engagement apps use the following token structure, that consists of a four character prefix and a 7-digit identifier.

 ![Tracking token structure.](../admin/media/tracking-token.png "Tracking token structure")  

 The following table lists tracking token parts and descriptions.  


|                  Part                   |                                                                                                                                                                                                                                                                                Description                                                                                                                                                                                                                                                                                |
|-----------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                 Prefix                  |                                                                                               Configurable from 1-20 characters. The default value is *CRM*: The prefix can be unique for each organization or environment. For example, in a multi-tenant deployment of customer engagement apps, we recommend that each organization configure and use a unique prefix.                                                                                               |
|                 Online/Offline designator                  |                                                                                               This is a legacy value used to indicate if the user was offline or online when sending the email. This digit is not configurable.                                                                                               |
|     Deployment base tracking number     |                                                                                                                                                                                 Configurable from 0-2,147,483,647. Default value is 0. Used as the base number for the user number digits. For example, if the value is 0 and 3 digits are configured for user number, the first user would have an identifier of 001. If the value is 500, the first user would be 501.                                                                                                                                                                                  |
|         Number of digits for user numbers         |                                                                                                                          Configurable from 1-10. The default range is three (3) digits. This value determines how many digits to use when customer engagement app generates the numeric identifier for the user who generated the email activity.                                                                                                                          |
| Number of digits for incremental message counter | Configurable from 1-9. Default range is three (3) digits. This value determines how many digits to use when customer engagement apps generates the numeric identifier for the email activity (not the individual messages that the activity contains). <br> **Note**: If you use the default value to generate a token with a three-digit number, it will increment the number through 999, and then restart the number at 000. You can use a larger order of digits to reduce the possibility of assigning duplicate tokens to active email threads.|

 To enable, disable, or configure tracking tokens, do the following::  

1.  Go to **Settings** > **Administration** > **System Settings**.  

2.  Click the **Email** tab.  

3.  In the **Configure email correlation** area you can disable, enable, or change the default tracking token structure.  

<a name="BKMK_smartmatching"></a>   

## Smart matching  
 When an incoming email message is processed by server-side sync, the system extracts information associated with the email message subject, sender address, and recipients' addresses that link the email activity to other rows. This correlation process, also known as smart matching, uses the following criteria to match received email message information to email activities:

- **Subject matching**: Prefixes, such as RE: or Re:, and letter case are ignored. For example, email message subjects with *Re: hello* and *Hello* would be considered a match..  
- **Sender and recipient matching**: The system calculates the number of exact sender and recipient email addresses in common.

When the matching process is complete, the system selects the owner and the object of the incoming email message.

By default, smart matching is turned off.  

> [!NOTE]
>  You can disable, enable, and modify smart-matching settings in the [System Settings dialog box – Email tab](system-settings-dialog-box-email-tab.md).  


### How does smart matching work

Smart matching relies completely on the existence of similarity between emails. The subject and recipients (from, to, cc and bcc) list are the two important components that are considered with checking for similarity.

When an email is sent from Dynamics 365, there is two sets of hashes that are generated and stored in the database.

- **Subject hashes**: To generate subject hashes, the subject of the email, which may include the Dynamics 365 token if its usage is enabled in system settings, is first checked for noise words like RE: FW: etc. The noise words are stripped off the subject and then tokenized. All the non empty tokens (words) are then hashed to generate subject hashes.

- **Recipient hashes**: To generate the recipient hashes the recipient (from, to, cc, bcc) list is analyzed for unique email addresses. For each unique email address an address hash is generated.

Next when an incoming email is tracked (arrived) in Dynamics 365, the same method is followed to create the subject and recipient hashes.

To find the correlation between the incoming email and the outgoing email the stored subject and recipient hashes are searched for matching values. Two emails are correlated if they have the same count of subject hashes and at least two matching recipient hashes.


> [!NOTE]
> As an organization sends and receives many emails over a long period of time, correlation data that is stored in the **EmailHash** table may increase significantly. This, when combined with less restrictive smart matching settings or common subject or recipient patterns, may cause smart matching performance to decrease. As a performance optimization, if high system resource utilization is detected when performing smart matching operations due to a combination of these factors, the system will correlate emails from the last 90 days.

### How can smart matching be configured?

There are advanced settings that allow you to manipulate the smart matching behavior.

-  **HashFilterKeywords**: This is a regular expression that is used to cancel out the noise in the subject line. All matching instances of the regular expression present in the subject line are replaced with empty strings before generating the subject hashes. <br/>**Default value**: ^\[\\s\]\*(\[\\w\]+\\s?:\[\\s\]\*)+

Basically it indicate that we internally (by default) will ignore any word at (multiples of it) at the start of the subject line that has a ":" at the end of it example:

|     | Subject  | Ignored words |
|-----|--------------|-------------------|
| **1**   | Test         | None              |
| **2**   | RE: Test     | RE:               |
| **3**  | FW: RE: Test | FW: RE:           |


> [!NOTE]
>  By default we do not ignore starting phrases in the subject line like "Out of office:" as this does not have the first word with the ":" next to it. For ignoring this phrase you can update the regular expression in the registry as "^\[\\s\]\*(\[\\w\]+\\s?:\[\\s\]\*)+\|Out of office:". Do not place the double quote that I have around the string in the example into the registry. The text in the registry should only be the regular expression you want to use for ignoring words from the subject line.

- **HashMaxCount**:This is the max number of hashes that are generated for any subject or recipient list. For example, if the subject after noise cancellation contains more than 20 words only the first 20 words are considered. <br> **Default value**: 20

- **HashDeltaSubjectCount**: This is the maximum delta allowed between subject hash counts of the emails to be correlated. <br> **Default value**: 0

- **HashMinAddressCount**: This is the minimum hash count matches required on the recipients list for the emails to be correlated. <br> **Default value**: 2



### See also  
 [Forward mailbox vs. individual mailboxes](../admin/forward-mailbox-vs-individual-mailboxes.md) <br>
 [Associate an email address with a row](../admin/associate-email-address.md)


[!INCLUDE[footer-include](../includes/footer-banner.md)]
