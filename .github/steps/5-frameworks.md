## User Journey: I want to orchestrate AI integrations using frameworks

> **Note**
> We recommend opening another browser tab to work through the following activities so you can keep these instructions open for reference.

[![Reset Journey](https://img.shields.io/badge/Reset--Journey-ff3860?logo=github)](/issues/new?title=Reset+Journey&labels=reset-journey&body=🔄+I+want+to+reset+my+AI+learning+journey+and+start+from+the+beginning.)

## Pre-requisites
1. A GitHub account
2. [Visual Studio Code](https://code.visualstudio.com/) installed
3. [Node.js](https://nodejs.org/en) installed
4. An Azure subscription. Use the [free trial](https://azure.microsoft.com/free/) if you don't have one, or [Azure for Students](https://azure.microsoft.com/free/students/) if you are a student.
5. [Azure Developer CLI](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd?tabs=winget-windows%2Cbrew-mac%2Cscript-linux&pivots=os-windows) installed

## Overview
In this step, you will learn how to orchestrate AI integrations using frameworks. [LangChain.js](https://js.langchain.com/docs/introduction/) is a framework for developing applications powered by language models. It provides a standard interface for working with different LLMs, tools, and data sources, making it easier to build complex applications.

## Step 1: Add LangChain.js to your project
We'll first install LangChain.js in our project to ensure our backend can communicate with Azure's LLM endpoints using LangChain's abstractions.

In your backend directory, run

```bash
npm install langchain @langchain/openai
```
The current backend code uses the Azure REST SDK directly. By switching to LangChain.js, we will decouple to code to take advantage of its abstractions and features like chains (composing tools and LLMs) and memory (storing conversation history).

### Update imports

Open `server.js` and replace:

```javascript
import ModelClient, { isUnexpected } from "@azure-rest/ai-inference";
import { AzureKeyCredential } from "@azure/core-auth";
``` 

with:

```javascript
import { AzureChatOpenAI } from "@langchain/openai";
```

### Update client initialization
Initialize LangChain's AzureChatOpenAI model client by replacing:

```javascript
const client = ModelClient(
  process.env.AZURE_INFERENCE_SDK_ENDPOINT,
  new AzureKeyCredential(process.env.AZURE_INFERENCE_SDK_KEY)
);
```

with:

```javascript
const chatModel = new AzureChatOpenAI({
  azureOpenAIApiKey: process.env.AAZURE_INFERENCE_SDK_KEY,
  azureOpenAIApiInstanceName: process.env.INSTANCE_NAME, // Find this in target url: https://<INSTANCE_NAME>.services...
  azureOpenAIApiDeploymentName: process.env.DEPLOYMENT_NAME, // i.e "gpt-4o"
  azureOpenAIApiVersion: "2024-08-01-preview", // Find this in target url: ...?<VERSION>
  temperature: 1,
  maxTokens: 4096,
});
```

### Update chat endpoint

Replace the Azure REST SDK logic in the try-catch block (_app.post("/chat")_) with the following code:

```javascript
  try {
    const response = await chatModel.invoke(messages);
    res.json({ reply: response.content });
  } catch (err) {
    console.error(err);
    res.status(500).json({
      error: "Model call failed",
      message: err.message,
      reply: "Sorry, I encountered an error. Please try again."
    });
  }
```

### Test the integration

Restart the server to confirm the changes are working. 

## Step 2: Add conversation memory

Currently, the chat model does not remember previous messages. For example, if you send a message like "Hey, you can call me Terry. What should I call you?"

Then ask the model "Quiz time. What's my name?". The model will not remember your name.

![Memory test](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/memory-test.png?raw=true)

To add memory in natural conversations, we will use LangChain's built-in memory modules - `ChatMessageHistory` and `ConversationSummaryMemory`. Conversation memory allows the AI to remember previous exchanges in a session, enabling more context-aware and coherent responses and LangChain.js provides built-in memory modules that make this easy to implement. With LangChain, you can implement stateful AI app experiences without manually managing chat logs, and you can easily switch between in-memory, Redis, or other storage options.


### How it would work in your app

Each user session (or conversation) maintains a history of messages.
When a user sends a new message, the backend includes previous exchanges from that session in the prompt.
The AI can reference earlier questions and answers, making the chat feel more natural and intelligent.

### Update imports

```javascript
import { BufferMemory } from "langchain/memory";
import { ChatMessageHistory } from "langchain/stores/message/in_memory";
```

### Set up session-based in-memory store

Create an object to store session histories, allowing you to maintain separate chat histories for different users or sessions.

```javascript
const sessionHistories = {};
```

### Add a helper function to get/create a session history

This utility function will check if a session history already exists for a given session ID. If it doesn't, it will create a new one.

```javascript
function getSessionMemory(sessionId) {
  if (!sessionMemories[sessionId]) {
    const history = new ChatMessageHistory();
    sessionMemories[sessionId] = new BufferMemory({
      chatHistory: history,
      returnMessages: true,
      memoryKey: "chat_history",
    });
  }
  return sessionMemories[sessionId];
}
```

### Update the chat endpoint

Lastly, we'll update the /chat handler to fetch session memory and load the chat history. Before sending the prompt to the model, we will add the chat history to the messages array so that the AI has context when generating the next reply. We'll then save the latest user message and model response to the session memory.

```javascript
app.post("/chat", async (req, res) => {
  const userMessage = req.body.message;
  const useRAG = req.body.useRAG === undefined ? true : req.body.useRAG;
  const sessionId = req.body.sessionId || "default";

  let sources = [];

  const memory = getSessionMemory(sessionId);
  const memoryVars = await memory.loadMemoryVariables({});

  if (useRAG) {
    await loadPDF();
    sources = retrieveRelevantContent(userMessage);
  }

  // Prepare system prompt
  const systemMessage = useRAG
    ? {
        role: "system",
        content: sources.length > 0
          ? `You are a helpful assistant for Contoso Electronics. You must ONLY use the information provided below to answer.\n\n--- EMPLOYEE HANDBOOK EXCERPTS ---\n${sources.join('\n\n')}\n--- END OF EXCERPTS ---`
          : `You are a helpful assistant for Contoso Electronics. The excerpts do not contain relevant information for this question. Reply politely: "I'm sorry, I don't know. The employee handbook does not contain information about that."`,
      }
    : {
        role: "system",
        content: "You are a helpful and knowledgeable assistant. Answer the user's questions concisely and informatively.",
      };

  try {
    // Build final messages array
    const messages = [
      systemMessage,
      ...(memoryVars.chat_history || []),
      { role: "user", content: userMessage },
    ];

    const response = await chatModel.invoke(messages);

    await memory.saveContext({ input: userMessage }, { output: response.content });

    res.json({ reply: response.content, sources });
  } catch (err) {
    console.error(err);
    res.status(500).json({
      error: "Model call failed",
      message: err.message,
      reply: "Sorry, I encountered an error. Please try again."
    });
  }
});
```

To test this, open the chat UI in your browser and send a message like "Hey, you can call me Terry. What should I call you?" and then ask "Quiz time. What's my name?". The model should remember your name.

![Memory test passed](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/memory-test-passed.png?raw=true)

### :keyboard: Activity: Push code to your repository from the codespace

1. Use the VS Code terminal to add the files to the repository:

   ```
   git add .
   ```

2. Next from the VS Code terminal stage and commit the changes to the repository:

   ```
   git commit -m "Using LangChain.js to orchestrate AI integrations"
   ```

3. Finally from the VS Code terminal push to code to the repository:

   ```
   git push

Wait about 20 seconds then refresh this page (the one you're following instructions from). [GitHub Actions](https://docs.github.com/en/actions) will automatically update to the next step.