# Lessons Learned

status of design doc:

<!-- ![draft status](https://img.shields.io/badge/Status-Draft-red) -->
<!-- ![review status](https://img.shields.io/badge/Status-Being%20Reviewed-yellow) -->
![finalized status](https://img.shields.io/badge/Status-Finalized-green)
<!-- ![archived](https://img.shields.io/badge/Status-Archived-lightgrey) -->

Authors: Maciej Czop

Reviewers: 

- Christine Hagyard [X, 2021]

## Overview  

The Performance Dashboard product is a collection of Key Performance Indicators that measure the user, service and policy impacts of decisions and changes to digital services.

This "Lessons Learned" document is meant to be a repository of institutional memory for hard-won insights and understanding around the data sets, data points, metrics and KPIs cultivated and created for the dashboard.

## Context

> The Digital Channel will deliver a dignified, user centric, responsive and integrated digital journey that delights, motivates and empowers people to help themselves.

By measuring the user, service and policy metrics across channels in BDM, the analytics combined with qualitative data and user feedback will clearly demonstrate the quality of service, the impact of change to services and policies, and will ensure success is well understood, measured and connected to policy objectives 

## Goals

- Create a Performance Dashboard MVP
- Identify KPIs for user, service and policy impacts
- Measure, benchmark and track metrics that affect digital channel outcomes

## Milestones 


`Start Date: June 1, 2020`

```
Milestone 1 - [ August 1, 2020 ] Create MVP Performance Dashboard
```

`End Date: TBD`


## Proposed Solution

The proposed solution can be split into three main parts.

1) User Signup: The components necessary to allow users to be able to sign up as testers with the user engagement site
2) Delete My Data: The components necessary to allow users to be able to delete their data at any point and time they choose
3) Researcher Data Access: The components necessary to allow researchers to be able to access the CSV extract from azure storage using their Active Directory credentials. 

### 1. User Signup

![Integrating Measurement across ESDC diagram](./Slide3.PNG)

User signup involves the user filling out a form on the React client application (using Next.JS) in which the submission sends a POST request to our NextJS application. A unique identifier called a Collision Resistant Unique Identifier (CUID) is generated and the record is created in the CosmosDB database. Once this record is created, a unique link is generated where the CUID is included as a query argument (eg, `https://alpha.service.canada.ca/validation?cuid=cjld2cyuq0000t3rmniod1foy`), which is sent to the user in an email, in the user's language of choice. After clicking the link, they will be marked as confirmed in the database and redirected to a thank you page. If users do not validate their email within 24 hours of registration, their record is removed from the database and will have to re-submit the form.

#### Why a NoSQL Database?

While it's definitely possible to use a traditional SQL DB here, a NoSQL database does not require a schema. Thus, we can modify, add, or delete fields on any user record without having to perform database migrations, which saves us some admin overhead. Our data is not relational in any way, (i.e. not more than one normalized table) thus we will ourselves time by not strictly defining a schema or worrying about data migrations.

#### What is a CUID and why are you using it?

A CUID is 25-character unique identifier that is collision-resistant across distributed systems — basically, it means it is unique. So in this way we are absolutely sure that each of our users have a unique identifier. 

#### Can people confirm other accounts by guessing CUIDs without access to the email address?

It's _extremely_ difficult to guess a CUID even with random guessing attacks! CUIDs were designed to be unique across distributed systems so the likelihood of 2 of the same being generated or 1 being guessed is nearly impossible. 

#### How do you prevent spam?

We will be using hcaptcha which is excellent and detecting bot behaviours based on movements of mouse and a myriad of other factors similar to Google's recpatcha but with more privacy. If a user is suspected of being a bot a challenge is presented to confirm they are not. We will also present a thank you page once a user submits, meaning that one would need to navigate back in order to resubmit. This should prevent a majority of spam attacks we may receive.

#### What about duplicate entries?

Only one unique record per email will be generated. If the same email is submitted twice, we will show an error.


### 2. Delete My Data

![user delete my data architecture and flow](./images/design-doc-004/deletemydata.png)

Users have a right to delete their data. How do we do this in a secure way such that we are sure that it is the actual user that wants to delete their data? 

The first step is for a user to fill and submit a form with their email. If this user exists in the database (their email matches that of a record), then the record is retrieved and a JavaScript Web Token (JWT) token with an expiry of 10 minutes and the user's CUID is generated and included in a unique link (eg, `https://alpha.service.canada.ca/users/delete?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJjdWlkIjoiY2RzZnNkZmRzZmRzZmRGUyJ9.nZEHRqlNw8CCGBUL18XZkDWaFYeC7cxq1y_mzq93Vhg`). This will be emailed to the user in their language of choice. Once a user clicks on this link an HTTP GET request will be sent to the Next.JS application. If the JWT is valid (meaning that it hasn't been altered and the token is not expired) then the CUID is extracted and if the record exists in the database with the CUID, the record is deleted.

#### What the heck is a JWT and secure are they?

[JavaScript Web Tokens](https://jwt.io/introduction) (or JWT) tokens, are cryptographically secure tokens that store structured information called claims. Claims are tamper-proof; if the contents are changed, token is invalidated. This means we can send around these tokens knowing for sure we generated them and they have not been altered. 

Within a JWT token there is information such as when the token expires and the user's CUID that we can set such that when the token expires it's no longer valid. Since this token is only included via a link sent by email, the expiry time is short enough, and the token can't be tampered with, we are confident that it is the user that wants to perform this action. JWTs are widely used across distributed web and mobile applications. 

### 3. Researcher Data Access (Interim)

![flow diagram of how a DECD administrator can upload verified user accounts into DECD sharepoint)](https://user-images.githubusercontent.com/2454380/126378733-d7145b50-e55d-461d-a5a3-ac5a4b5a6091.png)

For a period after launch, we will provide data to DECD researchers by pulling data from our Azure CosmosDB onto client devices of our DECD cloud admins, and then uploading the formatted data into Sharepoint. The steps involved are:

1. A DECD Researcher or the Product Owner requests the latest data from the user intake form
2. A DECD Cloud administrator with the mondoDB CLI installed on their client device runs a DB dump of the data in our Azure CosmosDB
3. The Administrator runs a script locally to iterate through the rows, only including those who have been verified
4. The script outputs a .CSV file, saved on the Administrator's device
5. The Administrator uploads the .CSV file (as an Excel file) to a Sharepoint location that DECD researchers have access to, overwriting the previous file, since this data is newer.
6. The Administrator then deletes the .CSV file from their client device.

[Our preferred future state](https://github.com/DTS-STN/Alpha-Site/wiki/Design-Doc-004-Screener-Intake-Process#4-researcher-data-access-future-state) describes a process where we would use a serverless function to generate updated CSVs periodically, and then researchers would be able to access the CSV without developer intervention. The future state process thus saves a number of steps for our Cloud Administrators. This is something we will pursue after the July 30th launch date.

#### Is it a risk to download data locally onto client devices?

Our data is rated as unclassified so there is no formal restriction blocking Administrators from downloading the data. The inconvenience here comes from adding extra steps to the process and introducing confusion (for example, if the wrong file is uploaded, then researchers may not have access to the most recent data). We will prioritize removing this inefficiency as we move forward.

### 4. Researcher Data Access (Future state)

![architecture and flow diagram of how a researcher can access user data](./images/design-doc-004/userresearch.png)

So now comes the question of how a researcher is supposed to access user records. We need a secure way to do it such that only those that have authority to access the data are able to do so. To solve this we will use our Azure cloud's integration with Active Directory as well as IAM rules on our resource group to only allow researchers read only access to a CSV extract generated once 24 hours via a serverless azure function.


