---
accreditations:
- indrapaul24
- DEFAULT
availability: VALIDATED
description: A connector that connects your SailPoint IIQ instance to Moveworks via
  Creator Studio.
difficulty_level: BEGINNER
fidelity: GUIDE
name: Sailpoint IdentityIQ
time_in_minutes: 20
---

# Introduction

**SailPoint IdentityIQ** is a prominent leader in enterprise identity governance solutions, providing businesses with the necessary tools to manage digital identities securely and effectively.

This guide will step you through creating a connector within Agent Studio to establish a stable connection to your SailPoint IIQ instance and make API calls to it. This guide has been organized into four main sections:

1. (Optional) Install the Moveworks for Sailpoint Plugin
2. Set up OAuth API Client
3. Test with Postman
4. Integrate with Agent Studio

# Prerequisites

- Sailpoint IIQ account with Admin privileges to create a new service account and setup API Authentication
- Your Sailpoint instance must be of version **8.1 or more** for our plugin to work correctly
- Your Sailpoint instance must either be deployed to Cloud or you should make the API endpoints available/accessible to Moveworks
- [Install Postman](https://www.postman.com/downloads/) for testing the API connection

# Set up SailPoint IIQ

Authentication with SailPoint’s API endpoints is done over an OAuth client whose capabilities are bound to the same permissions as our proxy user ([see Sailpoint’s docs](https://community.sailpoint.com/t5/IdentityIQ-Wiki/OAuth-2-0-client-credentials-as-a-token-based-protocol-for-API/ta-p/77630#toc-hId--1185039208)).

## (Optional) Step 1: Install the Moveworks for Sailpoint Plugin

On top of the capabilities that are supported via their native ReST APIs, to support additional abilities like pulling in identity and approvals information, and actioning on those approval requests, we have built a custom Sailpoint Plugin which installs a few extra APIs in your Sailpoint instance. If you are building a plugin that deals with either Identity or Approval Requests within SailPoint IIQ, we strongly suggest you to install the custom Sailpoint plugin and then utilize the APIs exposed for building your Agent Studio plugin.

All authentication with these endpoints is done over the same OAuth client as mentioned above.

> Please download the .ZIP file of the Moveworks Plugin before proceeding to configure.
> 
> [moveworks_plugin.zip](https://developer.moveworks.com/file-hosting/sailpoint/moveworks_plugin.zip)

1. Ensure that the plugin feature is enabled in IdentityIQ and that you have **System Administrator** or **Plugin Administrator** capabilities to install plugins.

2. Open the **Installed Plugins** page by selecting Plugins from the list under the gear icon.
    
    ![Installed Plugins](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/installed_plugins.png)
    

3. Click **New** to upload the plugin.
    
    ![New](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/new%20plugin.png)
    

4. **Click to upload your plugin**. A window dialog will appear. You can drag & drop our ZIP file from there.
    
    ![Upload](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/upload%20plugin.png)
    

5. Finish the plugin installation following the prompts in your Sailpoint Instance.

## Step 2: Set up OAuth API Client

1. Create an Identity for Moveworks. We recommend naming the account `svc.moveworks`.
    
    ![*Home Page → Create Identity*](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/create%20identity.png)

    *Home Page → Create Identity*

    
    ![*Fill out information on Create Identity Page*](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/identity%20fill%20up.png)
    
    *Fill out information on Create Identity Page*

2. Make sure the new service account has the appropriate user capabilities enabled. If you are installing the Moveworks for Sailpoint plugin, then make sure that the  `Moveworks Approvals Plugin Service Account` user capability is enabled.
    
    ![User cap](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/user%20capabilities.png)
    

3. Go to configure API Authentication.
    
    ![API Auth](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/API%20Auth.png)


4. Create a new API Client, setting the Proxy User to our service account.
    
    ![OAuth Client](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/new%20oauth%20client.png)


5. Note your `Client ID` and `Client Secret` - this will be required later to configure the Connector within Agent Studio

## Step 3: Test with Postman

Once you have all the required credentials from the above process, please move on to test the OAuth API and try to fetch the token.

1. [SailPoint IIQ OAuth Authentication API](https://community.sailpoint.com/t5/IdentityIQ-Wiki/OAuth-2-0-client-credentials-as-a-token-based-protocol-for-API/ta-p/77630#toc-hId--122537317): An API for SailPoint IIQ to request and receive Access token before invoking IdentityIQ or the Moveworks Plugin API.
    
    ```bash
    curl --location --request POST '{{sailpoint_url}}/identityiq/oauth2/token?grant_type=client_credentials' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --user '{{sailpoint_client_id}}:{{sailpoint_client_secret}}'
    ```
    
    Replace the `sailpoint_client_id` with the `Client ID` and `sailpoint_client_secret` with the `Client Secret` obtained from the previous step.
    
2. Import this request into Postman, replace the credentials and execute it. You should get a successful response and an access token.
    
    ![image.png](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/image.png)
    

## Step 4: Integrate with Agent Studio

1. In Agent Studio, create a new connector with the following configuration:
    - Base URL: `{{sailpoint_url}}`. This will be the base URL of your SailPoint instance. If you visit your SailPoint instance’s home page and the URL looks something like the following: `https://seri.company2482-poc.demohub.sailpointtechnologies.com:8080/identityiq/home.jsf`, the portion before `/identityiq` is your base URL.
    - Auth Config: `Oauth2`
    - Oauth2 Grant Type: `Client Credentials Grant`
    - Client ID: `{{sailpoint_client_id}}`
    - Client Secret: `{{sailpoint_client_secret}}`
    - Oauth2 Token Url: `{{sailpoint_url}}/identityiq/oauth2/token?grant_type=client_credentials`
    - Oauth2 Client Authentication: `OAuth 2.0 with Basic Auth Header set to client id and client secret`
    
    ![Screenshot 2024-09-17 at 11.27.30 AM.png](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/Screenshot_2024-09-17_at_11.27.30_AM.png)
    

2. Test your Connector by setting up a demo API action
    1. In Agent Studio, create a new Plugin.
        1. Click on Plugins > Actions tab
        2. Click on CREATE to create a new plugin
    2. Set up your API Connection to configure the API endpoint based on the following:
        1. Click on Use Existing Connector > select the Sailpoint connector that you just created > Click on Apply. This will populate the Authorization section of the API Editor
            
          ![image.png](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/image%201.png)
            
        2. Fill out the Endpoint URL: `/identityiq/rest/ping`
        
    3. Click on Test to check if the Connector setup was successful and expect a successful response as shown below
        
        ![Screenshot 2024-09-25 at 7.18.05 PM.png](Sailpoint%20IdentityIQ%20c7d45655365d4d25b30bd22674c5b910/Screenshot_2024-09-25_at_7.18.05_PM.png)
        

# Congratulations

You have successfully integrated SailPoint IIQ’s APIs with Agent Studio. This opens up a variety of automation and integration possibilities using your SailPoint IIQ instance.