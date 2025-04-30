# User Journey: I want to add a simple chat interface

> **Note**
> We recommend opening another browser tab to work through the following activities so you can keep these instructions open for reference.

[![Reset Journey](https://img.shields.io/badge/Reset--Journey-ff3860?logo=github)](/issues/new?title=Reset+Journey&labels=reset-journey&body=🔄+I+want+to+reset+my+AI+learning+journey+and+start+from+the+beginning.)

## Pre-requisites

1. A GitHub account
2. [Visual Studio Code](https://code.visualstudio.com/) installed
3. [Node.js](https://nodejs.org/en) installed
4. An Azure subscription. Use the [free trial](https://azure.microsoft.com/free/) if you don't have one, or [Azure for Students](https://azure.microsoft.com/free/students/) if you are a student.
4. [Azure Developer CLI](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd?tabs=winget-windows%2Cbrew-mac%2Cscript-linux&pivots=os-windows) installed

## Overview
In this step, you will learn how to add a simple chat interface to your AI application using [Vite](https://vitejs.dev/), a modern frontend build tool that provides a fast and efficient development experience. We will also use [lit](https://lit.dev/) to create simple web components for the chat interface.

## Step 1: Initialize a new Vite project

### Bonus: Introduction to Azure Developer CLI

The Azure Developer CLI (azd) is a command-line tool that simplifies the process of building, deploying, and managing applications on Azure. Instead of writing the code from scratch, you can use the Azure Developer CLI to quickly set up a project with the basic code.

In your current working directory, run the following command (at the root of your project) to initialize an AI Chat Interface app:

```bash
azd init -t juliamuiruri4/vite-chat-interface
```

This will initialize a new Vite project with the following folder structure:

```markdown
├─webapp/
│ ├─── index.html
│ ├─── package.json
│ ├─── src/
│ │    ├── main.js
│ │    ├── index.css
│ │    ├── components/
│ │    │    ├── chat.js
│ │    │    └── chat.css
│ │    └── utils/
│ │        └── chatStore.js
│ └─── public/
│      └── vite.svg
├─.azure/
├─infra/
│ ├─── main.bicep
│ ├─── main.parameters.json
│ └─── abbreviations.json
├─azure.yaml
├─ .gitignore
├─ README.md
```

- `webapp/`: contains the frontend code for the chat interface.
- `infra/`: contains the infrastructure code (bicep) for deploying the chat interface to Azure.
- `.azure/`: contains essential configurations for Azure.
- `azure.yaml`: a configuration file that defines each service in your application and maps them to the corresponding Azure resources defined in `infra`.

To run the application locally, 

```sh
cd webapp
npm install
npm run dev
```

Navigate to `http://localhost:5173` in your browser to see the chat interface.

![AI Chat Interface](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/vite-lit-ai-chat-interface.png?raw=true)

## Step 2: Add your AI model to the chat interface

First, let's update our folder structure to include a `webapi`. At the root of your project, create a new folder called `packages` and move the `webapp` folder into it. 

>**Note**
> If you are prompted to update imports for 'webapp', select `Yes`. 

Inside the `packages` folder, create a new folder called `webapi`. This will be the backend API for your chat interface.

Your project structure should now look like this:

```markdown
  .azure/
  infra/
  ├─packages/
  │ ├─── webapp/
  │ ├─── webapi/
  .gitignore
  azure.yaml
  README.md
```

### Expose an HTTP endpoint for your AI API

The `ai-foundry.js` file you created in the previous step is a script and cannot be called directly from the browser. To connect the chat interface to the AI model, we need to expose an HTTP endpoint that can be called from the frontend.

To do this, we will set up an Express.js API in the `webapi` folder.

### Initialize a Node.js project
In the `webapi` folder, run the following command to initialize a new Node.js project:

```bash
npm init es6 -y
```

This will create a new `package.json` file in the `webapi` folder.

### Install required dependencies
Run the following command to install the required dependencies:

```bash
npm install express cors dotenv @azure-rest/ai-inference @azure/core-auth
```

Move the `.env` file you created in the previous step into the `webapi` folder.

### Create an Express.js API
Create a new file called `server.js` in the `webapi` folder and add the following code:

<details> <summary>Click to expand the `server.js` code</summary>

```javascript
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import ModelClient from "@azure-rest/ai-inference";
import { AzureKeyCredential } from "@azure/core-auth";

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

const client = new ModelClient(
  process.env.AZURE_INFERENCE_SDK_ENDPOINT,
  new AzureKeyCredential(process.env.AZURE_INFERENCE_SDK_KEY)
);

app.post("/chat", async (req, res) => {
  const userMessage = req.body.message;
  const messages = [
    { role: "system", content: "You are a helpful assistant" },
    { role: "user", content: userMessage },
  ];

  try {
    const response = await client.path("chat/completions").post({
      body: {
        messages,
        max_tokens: 4096,
        temperature: 1,
        top_p: 1,
        model: "gpt-4o",
      },
    });
    res.json({ reply: response.body.choices[0].message.content });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Model call failed" });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`AI API server running on port ${PORT}`);
});
```
</details>

### Start the server
In the `webapi/package.json` file, add the following script to start the server:

```json
"scripts": {
  "start": "node server.js"
}
```

Run the following command to start the server:

```bash
npm start
```

Your API server should now be running on `http://localhost:3001`. 

### Update the chat app to call the API

Update the chat UI's API call in `webapp/src/components/chat.js` to call the new API endpoint. Replace the existing `_mockApiCall` function with the following code:

```javascript
async _mockAiCall(message) {
  const res = await fetch("http://localhost:3001/chat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ message }),
  });
  const data = await res.json();
  return data.reply;
}
```

Rename the `_mockAiCall` function to `_apiCall` and update the `sendMessage` method to call the `_apiCall` function instead of `_mockApiCall`.

## Step 3: Test the chat app

With the server running, navigate to `http://localhost:5173` in your browser. You should be able to send messages to the AI model and receive responses.

![AI Chat Interface AI Foundry](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ai-chat-interface-ai-foundry.png?raw=true)

## Step 4: Deploy to Azure

The project is already configured to deploy the webapp to Azure Static Web Apps. The `azure.yaml` file contains the configuration for the webapp:

```yaml
webapp:
    project: packages/webapp
    host: staticwebapp
    language: js
    dist: dist
    hooks:
      predeploy:
        windows:
          shell: pwsh
          run: npm run build
        posix:
          shell: sh
          run: npm run build
```

and we already have the bicep code (Intrastructure as Code) in `infra/main.bicep` 

```bicep
module webapp 'br/public:avm/res/web/static-site:0.7.0' = {
  name: 'webapp'
  scope: resourceGroup
  params: {
    name: webappName
    location: webappLocation
    tags: union(tags, { 'azd-service-name': webappName })
    sku: 'Standard'
  }
}
```
The webapi service is not yet configured in the `azure.yaml` file. To add the webapi service, add the following code to the `azure.yaml` file inside the `services` node:

```yaml
webapi:
    project: packages/webapp
    host: appservice
    language: js
```

We'll also need to add the bicep code for the webapi service in `infra/main.bicep`. Add the following code to the `main.bicep` file to create an App Service and App Service Plan for the webapi service:

```bicep
module serverfarm 'br/public:avm/res/web/serverfarm:0.4.1' = {
  name: 'appserviceplan'
  scope: resourceGroup
  params: {
    name: appServicePlanName
    skuName: 'B1'
  }
}

module webapi 'br/public:avm/res/web/site:0.15.1' = {
  name: 'webapi'
  scope: resourceGroup
  params: {
    kind: 'app'
    name: webapiName
    serverFarmResourceId: serverfarm.outputs.resourceId
  }
}
```

Declare the following parameters at the top of the `main.bicep` file to pass the names of the webapi and app service plan to the module:

```bicep
param webapiName string = 'webapi'
param appServicePlanName string = 'appserviceplan'
```

Update your output section at the end of the `main.bicep` file to include the following outputs:

```bicep
output WEBAPI_URL string = webapi.outputs.defaultHostname
```

To deploy the application, 
- Ensure you are logged in with `azd auth login`,
- Run `azd up` and enter an environment name (e.g., `build-a-thon`),
- Select your Azure subscription,
- Select a location for the resources.

  ![azd up](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/azd-up.png?raw=true)


