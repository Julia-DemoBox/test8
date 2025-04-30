<!--
  <<< Author notes: Step 4 >>>
  Start this step by acknowledging the previous step.
  Define terms and link to docs.github.com.
  TBD-step-4-notes.
-->

## User Journey: I want to create my first AI app with RAG

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
In this step, you will learn how to add RAG (retrieval-augmented generation) capabilities to your AI app. RAG allows your app to draw context and information from your data, making it more powerful and capable of answering questions based on the information you provide.

> **Note**
> To complete this step, you will need to get a sample dataset in any format (e.g., PDF, CSV, JSON) to work with. 
>
> For this example, we will use a [sample Contoso Electronics Employee Handbook PDF](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/employee_handbook.pdf) file. **You can bring any file of your choice**, but make sure it contains relevant information that you want your AI app to use for RAG.

Create a new folder `data` in the root of your project and place the sample dataset in it. To search and read your PDF, you will need to extract the text from it. You can use any PDF parser library of your choice, but for this example, we will use the `pdf-parse` library.

Open a terminal in your backend folder and run the following command to install the `pdf-parse` library:

```bash
npm install pdf-parse
```

## Step 1: Update your backend to support RAG

To enable Retrieval Augmented Generation (RAG) in your app, you need to enhance your backend so it can “ground” AI answers in your own documents—like your employee handbook PDF. This means the AI won’t just guess; it will look up relevant information from your handbook and use that to answer user questions.

What you will do in this step:
- **Load and split the PDF file into chunks** - Read the employee handbook PDF and break it into smaller, manageable text chunks. This makes it easier to search for relevant information later.
- **Search for relevant chunks based on the user's query** - When a user asks a question, your backend will scan all the chunks and find the ones most related to the question.
- **Augment the AI prompt** - The backend will send the user’s question along with the most relevant handbook chunks to your AI model (hosted on AI Foundry). The AI will use this context to generate a more accurate, trustworthy answer.


Open your server code `webapi/server.js` and add the following code:

<details> <summary>Click to expand the `server.js` code</summary>

```javascript
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import ModelClient, { isUnexpected } from "@azure-rest/ai-inference";
import { AzureKeyCredential } from "@azure/core-auth";
import fs from "fs";
import path from "path";
import { fileURLToPath } from 'url';
import { dirname } from 'path';
import pdfParse from 'pdf-parse/lib/pdf-parse.js';

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
const projectRoot = path.resolve(__dirname, '../..');
const pdfPath = path.join(projectRoot, 'data/employee_handbook.pdf'); // Update with your PDF file path

const client = ModelClient(
  process.env.AZURE_INFERENCE_SDK_ENDPOINT,
  new AzureKeyCredential(process.env.AZURE_INFERENCE_SDK_KEY)
);

let pdfText = null; // Cache for the PDF text
let pdfChunks = []; // Array to hold the chunks of text from the PDF
const CHUNK_SIZE = 800; // Define the chunk size for splitting the text i.e 800 characters per chunk

async function loadPDF() {
  if (pdfText) return pdfText;
  if (!fs.existsSync(pdfPath)) return "PDF not found.";
  const dataBuffer = fs.readFileSync(pdfPath); // Read the PDF file
  const data = await pdfParse(dataBuffer); // Parse the PDF file
  pdfText = data.text; // Extract the text from the PDF
  let currentChunk = ""; 
  const words = pdfText.split(/\s+/); 
  for (const word of words) {
    if ((currentChunk + " " + word).length <= CHUNK_SIZE) {
      currentChunk += (currentChunk ? " " : "") + word;
    } else {
      pdfChunks.push(currentChunk);
      currentChunk = word;
    }
  }
  if (currentChunk) pdfChunks.push(currentChunk);
  return pdfText;
}

function retrieveRelevantContent(query) {
  const queryTerms = query.toLowerCase().split(/\s+/) // Convert query to relevant search terms
    .filter(term => term.length > 3)
    .map(term => term.replace(/[.,?!;:()"']/g, ""));
  if (queryTerms.length === 0) return [];
  const scoredChunks = pdfChunks.map(chunk => {
    const chunkLower = chunk.toLowerCase(); 
    let score = 0; // Score chunks based on the number of matches
    for (const term of queryTerms) {
      const regex = new RegExp(term, 'gi');
      const matches = chunkLower.match(regex);
      if (matches) score += matches.length;
    }
    return { chunk, score };
  });
  return scoredChunks
    .filter(item => item.score > 0)
    .sort((a, b) => b.score - a.score)
    .slice(0, 3)
    .map(item => item.chunk);
}

app.post("/chat", async (req, res) => {
  const userMessage = req.body.message;
  const useRAG = req.body.useRAG === undefined ? true : req.body.useRAG;
  let messages = [];
  let sources = [];
  if (useRAG) {
    await loadPDF();
    sources = retrieveRelevantContent(userMessage);
    if (sources.length > 0) {
      messages.push({ 
        role: "system", 
        content: `You are a helpful assistant answering questions about the company based on its employee handbook. 
        Use ONLY the following information from the handbook to answer the user's question.
        If you can't find relevant information in the provided context, say so clearly.
        --- EMPLOYEE HANDBOOK EXCERPTS ---
        ${sources.join('\n\n')}
        --- END OF EXCERPTS ---`
      });
    } else {
      messages.push({
        role: "system",
        content: "You are a helpful assistant. No relevant information was found in the employee handbook for this question."
      });
    }
  } else {
    messages.push({
      role: "system",
      content: "You are a helpful assistant."
    });
  }
  messages.push({ role: "user", content: userMessage });

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
    if (isUnexpected(response)) throw new Error(response.body.error || "Model API error");
    res.json({
      reply: response.body.choices[0].message.content,
      sources: useRAG ? sources : []
    });
  } catch (err) {
    res.status(500).json({ error: "Model call failed", message: err.message });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`AI API server running on port ${PORT}`);
});
```
</details>


## Step 2: Update your frontend to show sources

Users will want to see the sources of the information used by the AI model to answer their questions. YOu'll update the chat UI to display the PDF excerpts used for each response. We'll add a toggle for 'Use Employee Handbook', (update with your file name), and when RAG is enabled, the sources will be displayed below the response.

Open the `webapp/src/components/chat.js` file and update the code to include the following changes:

<details> <summary>Click to expand the `chat.js` code</summary>

```javascript
import { LitElement, html } from 'lit';
import { loadMessages, saveMessages, clearMessages } from '../utils/chatStore.js';
import './chat.css';

export class ChatInterface extends LitElement {
  static get properties() {
    return {
      messages: { type: Array },
      inputMessage: { type: String },
      isLoading: { type: Boolean },
      isRetrieving: { type: Boolean },
      ragEnabled: { type: Boolean }
    };
  }

  constructor() {
    super();
    this.messages = [];
    this.inputMessage = '';
    this.isLoading = false;
    this.isRetrieving = false;
    this.ragEnabled = true; // Enable by default
  }

  createRenderRoot() {
    return this;
  }

  connectedCallback() {
    super.connectedCallback();
    this.messages = loadMessages();
  }

  updated(changedProps) {
    if (changedProps.has('messages')) {
      saveMessages(this.messages);
    }
  }

  render() {
    return html`
    <div class="chat-container">
      <div class="chat-header">
        <button class="clear-cache-btn" @click=${this._clearCache}> 🧹Clear Chat</button>
        <label class="rag-toggle">
          <input type="checkbox" ?checked=${this.ragEnabled} @change=${this._toggleRag}>
          Use Employee Handbook
        </label>
      </div>
      <div class="chat-messages">
        ${this.messages.map(message => html`
          <div class="message ${message.role === 'user' ? 'user-message' : 'ai-message'}">
            <div class="message-content">
              <span class="message-sender">${message.role === 'user' ? 'You' : 'AI'}</span>
              <p>${message.content}</p>
              ${this.ragEnabled && message.sources && message.sources.length > 0 ? html`
                <details class="sources">
                  <summary>📚 Sources</summary>
                  <div class="sources-content">
                    ${message.sources.map(source => html`<p>${source}</p>`)}
                  </div>
                </details>
              ` : ''}
            </div>
          </div>
        `)}
        ${this.isRetrieving ? html`
          <div class="message system-message">
            <p>📚 Searching employee handbook...</p>
          </div>
        ` : ''}
        ${this.isLoading && !this.isRetrieving ? html`
          <div class="message ai-message">
            <div class="message-content">
              <span class="message-sender">AI</span>
              <p>Thinking...</p>
            </div>
          </div>
        ` : ''}
      </div>
      <div class="chat-input">
        <input 
          type="text" 
          placeholder="Ask about company policies, benefits, etc..." 
          .value=${this.inputMessage}
          @input=${this._handleInput}
          @keyup=${this._handleKeyUp}
        />
        <button @click=${this._sendMessage} ?disabled=${this.isLoading || !this.inputMessage.trim()}>
          Send
        </button>
      </div>
    </div>
  `;
  }

  _toggleRag(e) {
    this.ragEnabled = e.target.checked;
  }

  _clearCache() {
    clearMessages();
    this.messages = [];
  }

  _handleInput(e) {
    this.inputMessage = e.target.value;
  }

  _handleKeyUp(e) {
    if (e.key === 'Enter' && this.inputMessage.trim() && !this.isLoading) {
      this._sendMessage();
    }
  }

  async _sendMessage() {
    if (!this.inputMessage.trim() || this.isLoading) return;
    
    const userMessage = {
      role: 'user',
      content: this.inputMessage
    };
    
    this.messages = [...this.messages, userMessage];
    const userQuery = this.inputMessage;
    this.inputMessage = '';
    this.isLoading = true;
    
    try {
      if (this.ragEnabled) {
        this.isRetrieving = true;
      }
      
      const response = await this._apiCall(userQuery);
      
      this.messages = [
        ...this.messages,
        { 
          role: 'assistant', 
          content: response.reply,
          sources: response.sources || []
        }
      ];
    } catch (error) {
      console.error('Error calling model:', error);
      this.messages = [
        ...this.messages,
        { role: 'assistant', content: 'Sorry, I encountered an error. Please try again.' }
      ];
    } finally {
      this.isLoading = false;
      this.isRetrieving = false;
    }
  }

  async _apiCall(message) {
    const res = await fetch("http://localhost:3001/chat", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ 
        message,
        useRAG: this.ragEnabled 
      }),
    });
    const data = await res.json();
    return data;
  }
}

customElements.define('chat-interface', ChatInterface);
```
</details>

In the above code, we added a toggle for "Use Employee Handbook" to enable or disable RAG. When RAG is enabled, the AI model will use the relevant excerpts from the PDF to answer the user's question. The sources are displayed below the response in a collapsible section.


Let's also add some styling to make the chat interface look better. Open the `webapp/src/components/chat.css` file and add the following styles:

<details> <summary>Click to expand the `chat.css` styling file</summary>

```css
/* Add these styles */

.rag-toggle {
  float: right;
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 0.9rem;
}

.system-message {
  background-color: #f8f9fa;
  font-style: italic;
  text-align: center;
  padding: 8px;
  border-radius: 10px;
}

.sources {
  margin-top: 8px;
  font-size: 0.85rem;
  cursor: pointer;
}

.sources summary {
  color: #0d6efd;
  font-weight: bold;
}

.sources-content {
  background-color: #f8f9fa;
  padding: 10px;
  border-radius: 4px;
  margin-top: 5px;
  max-height: 200px;
  overflow-y: auto;
  border-left: 3px solid #6c757d;
}
```
</details>

## Step 3: Test your app

Make sure both the webapp and webapi are running.

```bash
# In one terminal, run the webapi
cd webapi
npm start

# In another terminal, run the webapp
cd webapp
npm start
```
Open your browser to use the app, usually at `http://localhost:5123`. 

>**Note**
> Modify to match the data you have in your project.

### Test with RAG ON

1. Make sure the **"Use Employee Handbook" checkbox is checked**.
2. Ask a question related to the employee handbook, such as _"What is our company's mission statement?"_
   - The expected outcome is that the AI will respond with an answer based on the content of the employee handbook PDF, and the relevant excerpts will be displayed below the response.

    ![AI Foundry RAG with context](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ai-app-with-rag.png?raw=true)

3. Now ask a question not covered in the employee handbook, such as _"What's the company's stock price?"_
    - The expected outcome is that the AI will respond saying it doesn't have the information, and no excerpts will be displayed.

    ![AI Foundry RAG out of scope](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/ai-app-with-rag-outofscope.png?raw=true)

### Test with RAG OFF
1. **Clear chat and uncheck the "Use Employee Handbook" checkbox**.
2. Ask a question related to the employee handbook, such as _"What is our company's mission statement?"_
   - The expected outcome is that the AI will respond with a generic answer, and likely ask for more context, and no excerpts will be displayed.

    ![AI Foundry no RAG no context](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/no-rag-company.png?raw=true)

3. Now ask any general question, such as _"What is the capital of Morocco?"_
   - The expected outcome is that the AI will respond with the correct answer, and no excerpts will be displayed.

    ![AI Foundry no RAG general question](https://github.com/juliamuiruri4/JS-Journey-to-AI-Foundry/blob/assets/js-ai-journey-assets/no-rag-general.png?raw=true)

Notice how, with RAG enabled, the AI is strictly limited to the handbook and refuses to answer unrelated questions. With RAG disabled, the AI is more flexible and answers any question to the best of its ability.
   
trigger for next step is a commit adding a webapp and api that connects to AI Foundry using the Foundry JavaScript SDK and API to azure functions.

Wait about 20 seconds then refresh this page (the one you're following instructions from). [GitHub Actions](https://docs.github.com/en/actions) will automatically update to the next step.

### :keyboard: Activity: Push code to your repository from the codespace

1. Use the VS Code terminal to add the files to the repository:

   ```
   git add .
   ```

2. Next from the VS Code terminal stage and commit the changes to the repository:

   ```
   git commit -m "Added RAG"
   ```

3. Finally from the VS Code terminal push to code to the repository:

   ```
   git push

**Wait about 60 seconds then refresh your repository landing page for the next step.**