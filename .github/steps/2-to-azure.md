
# User Journey: I have a prototype. Let's move to Azure 

> **Note**
> We recommend opening another browser tab to work through the following activities so you can keep these instructions open for reference.

[![Reset Journey](https://img.shields.io/badge/Reset--Journey-ff3860?logo=github)](/issues/new?title=Reset+Journey&labels=reset-journey&body=🔄+I+want+to+reset+my+AI+learning+journey+and+start+from+the+beginning.)

## Pre-requisites

1. A GitHub account
2. [Visual Studio Code](https://code.visualstudio.com/) installed
3. [Node.js](https://nodejs.org/en) installed
4. An Azure subscription. Use the [free trial](https://azure.microsoft.com/free/) if you don't have one, or [Azure for Students](https://azure.microsoft.com/free/students/) if you are a student.

## Overview
In this step, you will learn how to deploy your AI model to Azure after testing it locally with GitHub Models.

## GitHub Models to Azure AI Foundry

1. On GitHub Models, click on **Use this model** and select **Language: JavaScript** and **SDK: Azure AI Inference  SDK**. 

   ![Use model](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/use-this-model-button.png?raw=true)

2. Under **Create a personal access token**, select **Get production key** and this will take you to ai.azure.com and prompt you to:-
   - Sign in to your Azure account
   - Select your Azure subscription 
   - Deploy model

      ![Select subscription](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ai-foundry-select-subscription.png?raw=true)

3. The model you selected will be pre-populated in the **Deployment name** field. You can optionally click on **Customize** to change the default configuration on deployment type, model version, tokens per minute (TPM) rate limit etc.

      ![Customize model for deployment](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/deploy-model.png?raw=true)

## AI Foundry VS Code extension
To continue working with your model VS Code, you will need to install the AI Foundry VS Code extension, which will allow you to deploy and manage your models in Azure AI Foundry without leaving your IDE.

1. Click on the **Extensions** icon in the left sidebar of Visual Studio Code and search for **Azure AI Foundry**.

2. Once installed, click on the **AI Foundry** icon in the left sidebar and click on **Set Default Project**. Select your project and expand the **Models** section. You should see your deployed model(s) listed there.

3. Click on the model name to open the model details view, where you can see the model's metadata, including the model version, deployment status, and TPM rate limit.

      ![List of models](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-list.png?raw=true)

4. Right click on your model and select **Open in Playground**. This will open a tab in VS Code with a chat Playground, where you can test your deployed model.

      ![AI Foundry playground](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ai-foundry-playground.png?raw=true)

5. You can also use the **Compare** feature to compare the performance of your deployed model with other models for manual evaluation. Once you are happy with the performance of your deployed model, right click on the model and select **Open Code File**, then:
   - Select **SDK**: Azure AI Inference SDK/ Azure OpenAI SDK
   - Select **Language**: JavaScript
   - Select **Authentication**: Key Authentication

   A new file will be created with the code to call your deployed model.  Ensusre you save the file as `ai-foundry.js` in your project folder. 

6. Create a `.env` file in your project folder and update it with the following environment variables:

   ```bash
   AZURE_INFERENCE_API_KEY=your_api_key
   AZURE_INFERENCE_SDK_ENDPOINT=your_api_endpoint
   ```
   > **Note**
   > You can copy the API key and endpoint either from the model details page or right click on the model and select **Copy API Key/ Copy Endpoint**.

   > **Important**
   > Create a `.gitignore` file in your project folder and add the `.env` file to it to prevent it from being pushed to GitHub. This is important for security reasons, as the `.env` file contains sensitive information such as your API key.

7. Run `npm install dotenv` to install the `dotenv` package, which will allow you to load environment variables from the `.env` file.

   Add the following code to the top of your `ai-foundry.js` file to load the environment variables:

   ```javascript
   import dotenv from 'dotenv';

   dotenv.config();
   ```

8. Finally, run the `ai-foundry.js` file and observe the output in the terminal. You should see the response from your deployed model.

   ![AI Foundry sample code](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/run-ai-foundry-sample.png?raw=true)


## ✅ Activity: Push `sample.js` code to your repository

1.  **Push your changes to the repository:** In the terminal, run the following commands to add, commit, and push your changes to the repository:

    ```bash
    git add .
    git commit -m "Connect to Azure AI Foundry"
    git push
    ```

2.  **Refresh the page:** After pushing your changes, **wait about 20 seconds then refresh your repository landing page. [GitHub Actions](https://docs.github.com/en/actions) will automatically update to the next step.**