
# User Journey: I want to build a local Gen AI prototype

> **Note**
> We recommend opening another browser tab to work through the following activities so you can keep these instructions open for reference.

> If you'd wish to reset your progress and select a different user journey, you can do so by clicking this button:
>
> [![Reset Progess](https://img.shields.io/badge/Reset--Progress-ff3860?logo=mattermost)](/issues/new?title=Reset+Journey&labels=reset-journey&body=🔄+I+want+to+reset+my+AI+learning+journey+and+start+from+the+beginning.%0A%0A**Please+click+on+Create+below,+then+wait+about+15+seconds.+Your+progress+will+be+reset,+this+issue+will+automatically+close,+and+you+will+be+taken+back+to+the+Welcome+step+to+select+a+new+user+journey.**)

## Pre-requisites

1. A GitHub account
2. [Visual Studio Code](https://code.visualstudio.com/) installed
3. [Node.js](https://nodejs.org/en) installed

## Overview
You will build a **local Gen AI prototype** using JavaScript and TypeScript. This prototype will allow you to experiment with different AI models, parameters, and prompts. You will also learn how to debug and iterate on your code, and add observability to your prototype.

## GitHub Models

GitHub Models is a FREE service that provides access to a variety of AI models from different providers and a playground to experiment with them. You can use these models to build your own AI applications (prototypes), or just to learn more about how AI works.

With GitHub Models, you can:

  - Use GitHub Personal Access Tokens (PAT) to authenticate and access the models locally.
  - Use a single API key to access all the models.

1. Open [GitHub Models](https://github.com/marketplace/models)

2. Click on **explore the full model catalog** to see the available models.

    ![GH Models full catalog](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ghm-full-catalog.png?raw=true)

    You will see a broad range of models listed in the catalog.

    _But which model should I use for what?_

3. Scroll down to the **Filter section** to see the available filters. You can filter the models by:
    - **Publisher:** Cohere, DeepSeek, Meta, Mistral AI, Microsoft _(research)_, Azure OpenAI Service, and more.
    - **Category:** Conversation _(models optimized for dialogue use cases)_, Agents, Multimodal _(models capable of processing input in multiple formats - audio, visual etc.)_, Reasoning, and more.

    Select a model from the list to open the model card. The model card provides detailed information about the model, and may include:
    
    ### A README
    - **Model Abstract:** A brief description of the model and its capabilities.
    - **Model Architecture:** The data used to train the model and their modalities for input and output _(text-image pairs)_, the model size _(parameters)_, model context length _(how much text the model can process at once)_, training date _(knowledge cut-off date/data freshness for the model)_, supported languages,  and more.
  
      ![Model Architecture](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-readme.png?raw=true)

    ### A Transparency Note
    - **Model Use cases:** Primary and out-of-scope use cases for the model, responsible AI considerations, content filtering configurations and more.
  
      ![Model Transparency notice](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-transparency-notice.png?raw=true)

    ### A License 
    - **Model License:** The license under which the model is released, including any restrictions on use or distribution.

      ![Model License notice](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-license.png?raw=true)

    ### An Evaluation Report
    - **Model Benchmarks:** A summary of the model's performance on various benchmarks, including accuracy, speed, and other relevant metrics.
    - **Model Benchmarks:** A list of benchmarks used to evaluate the model's performance, including details on how the model performed on each benchmark. Metrics may include:
      - MMLU Pass@1 (Measuring Massive Multitask Language Understanding) - Knowledge and reasoning across science, math, and humanities.
      - DROP - Measures reading comprehension and numerical reasoning capabilities
      - among others

      ![Model Evaluation](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-evaluation.png?raw=true)

4. After selecting a model and reviewing the model card, you can use the **Playground** to experiment with the model. The playground provides a user-friendly interface for testing the model's capabilities and understanding how it works.

    ![Playground button](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-playground-button.png?raw=true)

    You can directly send questions (prompts) to the model and see how it responds. Throughout the session, you can monitor the token usage and the model's response time at the top of the chat UI.

    ![Playground token usage note](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/playground-token-usage.png?raw=true)

5. To check your token usage against your GitHub Models free quota (input/ output tokens, latency), click on the **Input: Output: Time** note at the top right of the chat UI to open Model usage insights.

    ![Playground token usage card](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-usage-insights.png?raw=true)

6. Before going further, on the right side of the playground, switch from **Details** to **Parameters** to see the available parameters that you can adjust to change the model's behavior. 

    ![Playground parameters](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/model-parameters.png?raw=true)

    The parameters include:
    - **Max Tokens:** The maximum number of tokens the model can generate in response to a prompt. Adjusting this parameter can help control the length of the model's output. 
    - **Temperature:** Controls the randomness of the model's output. A higher temperature (e.g., 0.8) makes the output more random, while a lower temperature (e.g., 0.2) makes it more focused and deterministic.
    - **Top P:** Controls the diversity of the model's output. A higher value (e.g., 0.9) allows for more diverse outputs, while a lower value (e.g., 0.1) makes the output more focused on the most likely tokens.
    - **Penalty Penalty:** Controls the model's tendency to repeat itself. A higher value (e.g., 1.0) discourages repetition, while a lower value (e.g., 0.0) allows for more repetition.
    - **Frequency Penalty:** Similar to the penalty penalty, this parameter controls the model's tendency to repeat the same words or phrases. 
    - **Stop:** A list of tokens that, when generated, will stop the model's output. 

    You can continue interacting with the model in the playground, as you adjust the parameters to ensure you get the desired output.

    _How do models compare across different prompts and parameters?_

7. GitHub Models provides a **Compare** feature that allows you to compare the performance of different models on the same prompt. This is useful for understanding how different models respond to the same input and can help you choose the best model for your specific use case.

    Click on the **Compare** button at the top right of the playground.

    ![Compare](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/compare-button.png?raw=true)

    Select the models you want to compare from the list of available models.

    This will open a chat UI for the selected models side by side, and your prompt will be sent to both models. 

    ![Compare chat example](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/compare-chat.png?raw=true)

    In the example provided, you can compare the performance of a reasoning model and a conversation model on the same prompt to understand their strengths and limitations.

## Playground to VS Code

Now that you have a better understanding of the GitHub Models playground, let's look at how to use the models in your own JavaScript code.

1. On the far right, click on **Use this model** and select **Language: JavaScript** and **SDK: Azure AI Inference  SDK**. 

    ![Use model](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/use-this-model-button.png?raw=true)

    Follow the instructions provided to:
    - Get a free developer key, (Personal Access Token (classic)) and store it in an environment variable either using bash, PowerShell or command line.
    - Install dependencies 
    - Run basic code sample. **Ensure your local file is named `sample.js`**

      ![Run node sample file](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/node-sample.png?raw=true)

## Exercise: Convert a hand-drawn sketch to a web page

1. Download the contoso website hand-drawn sketch from [this link](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/contoso_layout_sketch.jpg), and save it `contoso_layout_sketch.jpg` in the same directory as your `sample.js` file.

    > **Note**
    > If you aren't using a muiltimodal model, swap out the `modelName` in the code sample with a multimodal model of your choice. You can find a list of multimodal models in the GitHub Models catalog.

  2. Update the code to pass the image to the model as input.

      > **Note**
      > You can use [GitHub Copilot](https://github.com/features/copilot) to help you with this task.
      >
      > ![Update code with GitHub Copilot](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/github-copilot.png?raw=true)

  3. Run the code and check the output in the console.

      ![Run sample passing image](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/run-sample-for-code.png?raw=true)

## Use AI Toolkit in VS Code

The AI Toolkit in Visual Studio Code is a powerful extension that provides a set of tools and features to help you build AI applications more efficiently. 

1. Click on the **Extensions** icon in the left sidebar of Visual Studio Code and search for **AI Toolkit**.

2. Similar to GitHub Models, with the AI Toolkit now installed, you can browse through the catalog of available models, and use the **Playground** to experiment with the models, all on VS Code.

    ![AI Toolkit catalog](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ai-toolkit.png?raw=true)

    Let's execute the task above using the AI Toolkit in VS Code.

3. Select a multimodal model from the catalog and open the **Playground**.

    In the playground, upload the `contoso_layout_sketch.jpg` image and enter a prompt to write the HTML code for the website.

4. On the generated code, click on the **New file** icon to copy the generated code into a new file. Save it as `index.html` in the same directory as your `sample.js` file.

    ![AI Toolkit -html](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ai-toolkit-html.png?raw=true)

    Do the same for the CSS code and save it as `style.css` in the same directory.

    You can preview the generated code and iterate on the code to improve it (optionally using GitHub Copilot).

    ![AI Toolkit - html preview](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/html-preview.png?raw=true)

## ✅ Activity: Push `sample.js` code to your repository

>**Note**
> To complete this user journey and **AUTOMATICALLY UPDATE** your progress, you MUST push your code to the repository as described below.
>
> Checklist:
> - [ ] Have a `sample.js` file at the root of your project
> - [ ] The file MUST include a reference to your GITHUB_TOKEN environment variable

> If you'd wish to jump to another step without completing this user journey, you can do so by hitting this button:
>
> [![Skip to another journey](https://img.shields.io/badge/Skip--to--another--journey-ff3860?logo=mattermost)](/issues/new?title=Skip+Journey&labels=reset-journey&body=🔄+I+want+to+reset+my+AI+learning+journey+and+start+from+the+beginning.%0A%0A**Please+click+on+Create+below,+then+wait+about+15+seconds.+Your+progress+will+be+reset,+this+issue+will+automatically+close,+and+you+will+be+taken+back+to+the+Welcome+step+to+select+a+new+user+journey.**)

1.  **Push your changes to the repository:** In the terminal, run the following commands to add, commit, and push your changes to the repository:

    ```bash
    git add .
    git commit -m "Working with GitHub Models and AI Toolkit"
    git push
    ```

2.  **Refresh the page:** After pushing your changes, **wait about 20 seconds then refresh your repository landing page. [GitHub Actions](https://docs.github.com/en/actions) will automatically update to the next step.**
