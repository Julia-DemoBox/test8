

## User Journey: I want to build an AI Agent

> **Note**
> We recommend opening another browser tab to work through the following activities so you can keep these instructions open for reference.

[![Reset Journey](https://img.shields.io/badge/Reset--Journey-ff3860?logo=github)](/issues/new?title=Reset+Journey&labels=reset-journey&body=🔄+I+want+to+reset+my+AI+learning+journey+and+start+from+the+beginning.)

## Pre-requisites
1. A GitHub account
2. [Visual Studio Code](https://code.visualstudio.com/) installed
3. [Node.js](https://nodejs.org/en) installed
4. An Azure subscription. Use the [free trial](https://azure.microsoft.com/free/) if you don't have one, or [Azure for Students](https://azure.microsoft.com/free/students/) if you are a student.
5. [Azure Developer CLI](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd?tabs=winget-windows%2Cbrew-mac%2Cscript-linux&pivots=os-windows) installed
6. AI Foundry VS Code Extension installed

## Overview

In this step, you will learn how to build a basic AI agent using the AI Foundry VS Code extension. An AI agent is a program powered by AI models that can understand instructions, make decisions, and perform tasks autonomously. We'll start with a simple agent and test its capabilities.

## Step 1: Create an Agent using the AI Foundry Extension

1.  **Open the AI Foundry Extension:** Click on the AI Foundry icon in the Activity Bar on the left side of VS Code.
2.  **Create New Agent:** Ensure your default AI Foundry project is selected. Hover over the "Agents" section title and click the "+" (Create Agent) icon that appears.

    ![Create Agent Button](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/create-agent.png?raw=true)

4.  **Name the Agent Configuration:** You'll be prompted to name the agent's configuration file. Enter `my-agent.yaml` and press Enter. This file will store your agent's definition.
5.  **Select a Base Model:** Choose a foundation model for your agent. For this example, select a model from your model's list. This model will power the agent's core reasoning and language capabilities.
6.  **Define Initial Agent Preferences:** The Agent Designer will open up as a new file (or you can right click on the `.yaml` file and select `Open Agent Designer`. You'll be prompted to provide:-
    - A model name: _Example. gpt-4o-mini_
    - System instructions for your agent. This tells the agent how it should behave. Enter the following:

      ```
      You are a helpful agent who loves emojis 😊. Be friendly and concise in your responses.
      ```
    - Edit temparature: 0.7 
      ````yaml
      version: 1.0.0
      name: my-agent
      description: Description of the agent
      id: ''
      model:
        id: gpt-4o-mini
        options:
          temperature: 0.7
          top_p: 1
      instructions: >-
        You are a helpful agent who loves emojis 😊. Be friendly and concise in your
        responses.
      tools: [] # We'll add tools later
      ````
7.  **Deploy to Azure AI Foundry:** Click on the "Deploy to Azure AI Foundry" button in the bottom left corner of the Agent Designer. This will deploy your agent to Azure AI Foundry, and it will pop up in the extension under the "Agents" section.

    ![Deploy to Azure AI Foundry Button](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/deploy-to-ai-foundry.png?raw=true)

## Step 2: Test the Agent in the Playground
1.  **Open the Playground:** Right-click on the agent you just created in the "Agents" section and select "Open Playground". This will open a new tab where you can interact with your agent. 

    Alternatively, you can expand the "Tools" section and click on "Agent Playground", then select your agent from the list.

    ![Agent Playground in tools](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/agent-playground-in-tools.png?raw=true)

2.  **Test the Agent:** In the Playground, you can start chatting with your agent. Try sending it a few messages to see how it responds. For example:
    - "Hi there!" 
    - Expect a friendly response with emojis.

      ![Agent Playground](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/agent-playground.png?raw=true)

    - "What's the weather in Nairobi right now?"
    - Expect a response like "I don't know the weather right now, but you can check a weather website for the latest updates! 🌤️"

      ![Agent Playground - weather response](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/agent-weather-response.png?raw=true)

The agent currently has some limitations including:
- No access to real-time data or external APIs.
- It can't perform complex tasks or make decisions beyond simple responses.
- It may not always provide accurate or relevant information.

So in the next step, we will add a tool to the agent to make it more useful.

## Step 3: Add a Tool to the Agent

Tools calling/ function calling is a powerful feature that allows your agent to perform specific tasks or access external data. In this step, we will add a tool to our agent that can fetch the current weather information.

Create a new file called `bing.yaml` in the same directory as your agent configuration file. This file will define the tool's configuration.

```yaml
type: bing_grounding
id: bing_grounding
options:
  tool_connections:
    - <tool_connection_id>
```

Create a bing resources service on the azure portal and add the api key to the agent yaml file. 

On the Agent Designer, click on the + icon next to the "Tools" section. This will prompt you to select a yaml file for the tool. Select the `bing.yaml` file you created earlier. This file contains the configuration for the Bing Search API tool.

Click on the "Deploy to Azure AI Foundry" button in the bottom left corner of the Agent Designer. This will deploy your updated agent with the new tool to Azure AI Foundry.

![Add bing tool](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/add-bing-tool.png?raw=true)

## Step 4: Test the Agent with the Tool

Now that we have added the Bing Search API tool to our agent, we can test it in the Playground. Open the "Agent Playground" and send the agent a message like "What's the weather in Nairobi right now?" The agent should use the Bing Search API tool to fetch the current weather information and respond with a friendly message.

![Weather with Bing](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/weather%20-with-bing.png?raw=true)


## ✅ Activity: Push code to your repository from the codespace

1.  **Push your changes to the repository:** In the terminal, run the following commands to add, commit, and push your changes to the repository:

    ```bash
    git add .
    git commit -m "Add AI agent with Bing Search API tool"
    git push
    ```
2.  **Wait for the GitHub Actions to run:** After pushing your changes, GitHub Actions will automatically trigger a workflow to build and deploy your agent to Azure AI Foundry. You can check the progress of the workflow in the "Actions" tab of your repository.
3.  **Refresh the page:** Wait about 20 seconds then refresh this page (the one you're following instructions from). [GitHub Actions](https://docs.github.com/en/actions) will automatically update to the next step.

