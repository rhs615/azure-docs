---
title: include file
description: include file
services: azure-communication-services
author: gistefan
manager: soricos

ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 10/08/2021
ms.topic: include
ms.custom: include file
ms.author: gistefan
---

## Set up prerequisites

- [Node.js](https://nodejs.org/) Active LTS and Maintenance LTS versions (8.11.1 and 10.14.1 recommended).

## Set up

### Create a new Node.js Application

Open your terminal or command window create a new directory for your app, and navigate to it.

```console
mkdir communication-access-tokens-quickstart && cd communication-access-tokens-quickstart
```

Run `npm init -y` to create a `package.json` file with default settings.

```console
npm init -y
```

### Install the package

Use the `npm install` command to install the Azure Communication Services Identity SDK for JavaScript.

```console

npm install @azure/communication-identity@beta --save
npm install @azure/msal-node --save
npm install express --save

```

The `--save` option lists the library as a dependency in your **package.json** file.

## Set up the app framework

From the project directory:

1. Open a new text file in your code editor
1. Add a `require` call to load the `CommunicationIdentityClient`
1. Create the structure for the program, including basic exception handling

    ```javascript
    const { CommunicationIdentityClient } = require('@azure/communication-identity');
    const express = require("express");
    const msal = require('@azure/msal-node');
    
    const SERVER_PORT = process.env.PORT || 80;
    const REDIRECT_URI = "http://localhost"; 
    
    // Quickstart code goes here
    
    app.listen(SERVER_PORT, () => console.log(`Communication access token application started on ${SERVER_PORT}!`))
    
    ```

1. Save the new file as `issue-communication-access-token.js` in the `access-tokens-quickstart` directory.

### Step 1: Receive the Azure AD user token via the MSAL library

The first step in the token exchange flow is getting a token for your Teams user by using [Microsoft.Identity.Client](../../../active-directory/develop/reference-v2-libraries.md).

```javascript
const msalConfig = {
    auth: {
        clientId: "<contoso_application_id>",
        authority: "https://login.microsoftonline.com/<contoso_tenant_id>",
    }
};

const pca = new msal.PublicClientApplication(msalConfig);
const provider = new msal.CryptoProvider();

const app = express();

app.get('/', async (req, res) => {
    const {verifier, challenge} = await provider.generatePkceCodes();
    const authCodeUrlParameters = {
        scopes: ["https://auth.msft.communication.azure.com/VoIP"],
        redirectUri: REDIRECT_URI,
        codeChallenge: challenge, 
        codeChallengeMethod: "S256"
    };

    pca.getAuthCodeUrl(authCodeUrlParameters).then((response) => {
        res.redirect(response);
    }).catch((error) => console.log(JSON.stringify(error)));
});

app.get('/redirect', async (req, res) => {
    const tokenRequest = {
        code: req.query.code,
        scopes: ["https://auth.msft.communication.azure.com/VoIP"],
        redirectUri: REDIRECT_URI,
    };

    pca.acquireTokenByCode(tokenRequest).then((response) => {
        console.log("Response: ", response);
        //TODO: the following code snippets go here
        res.sendStatus(200);
    }).catch((error) => {
        console.log(error);
        res.status(500).send(error);
    });
});
```

### Step 2: Initialize the CommunicationIdentityClient

Instantiate a `CommunicationIdentityClient` with your connection string. The code below retrieves the connection string for the resource from an environment variable named `COMMUNICATION_SERVICES_CONNECTION_STRING`. Learn how to [manage your resource's connection string](../create-communication-resource.md#store-your-connection-string).

Add the following code to the `then` method:

```javascript
// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the identity client
const identityClient = new CommunicationIdentityClient(connectionString);
```

### Step 3: Exchange the Azure AD user token for the Teams access token

Use the `getTokenForTeamsUser` method to issue an access token for the Teams user that can be used with the Azure Communication Services SDKs.

```javascript
let accessToken = await identityClient.getTokenForTeamsUser(teamsToken);
console.log(`Token: ${accessToken}`);
```

## Run the code

From a console prompt, navigate to the directory containing the *issue-communication-access-token.js* file, then execute the following `node` command to run the app.

```console
node ./issue-communication-access-token.js
```
