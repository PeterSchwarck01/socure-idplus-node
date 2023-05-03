<!--
 * The contents of this file are subject to the terms of the Common Development and
 * Distribution License (the License). You may not use this file except in compliance with the
 * License.
 *
 * You can obtain a copy of the License at legal/CDDLv1.0.txt. See the License for the
 * specific language governing permission and limitations under the License.
 *
 * When distributing Covered Software, include this CDDL Header Notice in each file and include
 * the License file at legal/CDDLv1.0.txt. If applicable, add the following below the CDDL
 * Header, with the fields enclosed by brackets [] replaced by your own identifying
 * information: "Portions copyright [year] [name of copyright owner]".
 *
 * Copyright 2022 ForgeRock AS.
-->

# Socure ID+ Identity Verification Nodes

Learn how to install and configure the Socure ID+ Identify Verification nodes for ForgeRock's [Identity Platform](forgerock_platform) 7.2.0 and later. 

## Overview 

Socure’s revolutionary ID+ Platform utilizes every element of identity, natively orchestrated by advanced AI and ML, to maximize accuracy, reduce false positives, and eliminate the need for disparate products. 

After you complete the build and configuration process with ForgeRock, the following 2 nodes will be available:

1. **SocureId+ Authentication Node**: Verifies the collected user attributes using Socure's ID+ API and returns a decision for the user identity.
2. **Socure Predictive Docv Node**: Verifies a user's identity by authetnicating their government-issued ID. The collected data is then returned in the response, along with the Predictive Document Verification (DocV) results. 

## Installation 

1. Copy the `.jar` file from the `../target` directory to the `../web-container/webapps/openam/WEB-INF/lib` directory where AM is deployed. 
2. Restart the web container for the new node to become available. The node will then appear in the Authentication Tree components palette.

## Use cases

The Socure integration with ForgeRock supports both identity verification and identity proofing using the workflows in the sections below. 

![ID+ workflow](./screenshots/usecase1.png)

### Use case: SocureId+ Authentication node

To verify a user's identity with the Socure's ID+ API, your Authentication Tree should be configured as follows: 

1. The Page Node gathers the attributes required to verify the user identity. 
2. The SocureId+ node verifies the identity using the ID+ API and returns a simple decision outcome (reject, refer, resubmit, review, or accept) for the user identity. 
    - If the decision is accept, the Create Object creates a resource with the information gathered by the previous nodes. The user is then automatically logged in after their identity is verified and the flow is successful. 
    - If the decision is refer, the Socure Predictive DocV node can be initiated for a step up in authentication. See **Use case: Socure's Predictive Docv Node** below for more information. 

### Use case: Socure Predictive Docv Node

In use cases that require a step up in authentication, Socure Predictive DocV node can be used as an additional authentication source to authenticate the user's ID and verify their, then recommend if they should be accepted or rejected.

When implementing Socure's Predictive DocV node, your Authentication Tree should be configured as follows: 

1. The Page Node gathers the attributes required to verify the user identity. 
2. The SocureId+ node verifies the identity using the ID+ API and returns a simple decision outcome (reject, refer, resubmit, review, or accept) for the user identity. 
3. If the decision is refer, the SocureId+ node will initiate a step up authentication workflow using the Socure Predictive Document Verification (DocV) node and the DocV Web SDK. 
4. The user follows the instructions in the DocV workflow to authenticate their government-issued ID and verify their identity. After the flow completes successfully, the Create Object will create a resource with the information gathered by the previous nodes. The user is then automatically logged in after their identity is verified and the flow is successful.

## Configuration

The code in this repository has binary dependencies that live in the ForgeRock Maven repository. Maven can be configured to authenticate to this repository by following the steps in the [ForgeRock Knowledge Base Article](https://backstage.forgerock.com/knowledge/kb/article/a74096897).

### Prerequisites

Before completing the configuration steps in the sections below, you will need to create a Socure account to retrieve your API key. 

#### Create an account

1. Go to [Admin Dashboard](dashboard.socure.com).
2. Click **Create Account**.
3. Enter the requested information in the fields provided.
4. Click **Submit**. 

New accounts must be approved by Socure. Once approved, you will receive an email with a link to set your account password. The link will expire after 7 days.

#### Configure your account

1. Log in to [Admin Dashboard](dashboard.socure.com).
2. On the **Account/Environment** menu, select **Production**, **Certification**, or **Sandbox**.
3. Go to **Developers > IPs & Domains**, then click **API Access**.
4. Click **New Domain**, then enter the IP address or domain in the **IP/Domains** field.
5. Click **Create**.

#### Retrieve your API Key

1. Log in to [Admin Dashboard](dashboard.socure.com).
2. Select an environment from the **Account/Environment** menu on the upper-right corner of the page.
3. Go to **Developers > ID+ Keys**, then click the vertical ellipsis and select **Copy Key**.

Socure provides a set of API keys for the Sandbox, Certification, and Production environments. Users must have the correct permissions enabled for their account to access their keys.

### Configure SocureId+ Node

Verifies the collected user attributes using Socure's ID+ API and returns a decision for the user identity.

![Configure SocureId+ node](./screenshots/socureIdPlusConfig.png)
![Attributes](./screenshots/attributes.png)

| Configuration     | Description                                                                                                                                                   | Example                                                                                             |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| Name              | A name for the node.                                                                                                                                          | SocureId+ Node                                                                                      |
| SocureAPIEndpoint | The API URL path.                                                                                                                                             | https://sandbox.socure.com/api/3.0/EmailAuthScore                                                   |
| SocureAPIKey      | Your API key for the environment you want to access. You can find your SDK key in Admin Dashboard.                                                            | SocureApiKey a1b234cd-xxxx-xxxx-xxxx-56abcdef6789                                                   |
| modules           | A configurable list of ID+ modules that are called in the ID+ API call.                                                                                       | emailrisk, phonerisk, fraud, addressrisk, synthetic, decision, kyc                                  |
| attributes        | Maps ForgeRockLDAP Attributes to ID+ API. The KEY should be the Socure attribute JSON key and the VALUE should be the corresponding ForgeRock LDAP Attribute. | streetAddress, city, zipCode, countryCode, email, ssn, lastName, dob, firstName, mobilePhone, state |

### Configure Socure Predictive Docv node

Verifies a user's identity by authetnicating their government-issued ID. The collected data is then returned in the response, along with the Predictive Document Verification (DocV) results. 

![ScreenShot](./screenshots/docv_config.png)

| Configuration             | Description                                                                                        | Example                                           |
|-------------------|----------------------------------------------------------------------------------------------------|---------------------------------------------------|
| Name              | A name for the node.                                                                               | Socure Predictive DocV                            |
| SocureAPIEndpoint | The API URL path.                                                                                  | https://sandbox.socure.com/api/3.0/EmailAuthScore |
| SocureAPIKey      | Your API key for the environment you want to access. You can find your SDK key in Admin Dashboard. | SocureApiKey a1b234cd-xxxx-xxxx-xxxx-56abcdef6789 |
| isDocVRequired    |                                                                                                    | true                                              |
| websdkUrl         | The URL for the latest version of the DocV Web SDK.                                                | https://websdk.socure.com/bundle.js               |
