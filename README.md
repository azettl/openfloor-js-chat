
# Building a Simple Client with JavaScript

In this guide, we'll create a simple single-file HTML chat interface to test your Open Floor agents. This demo will let you fetch agent manifests and have conversations with any [Open Floor Protocol](https://openfloor.dev/introduction)-compliant agent. We'll build everything using vanilla HTML, CSS, and JavaScript with the help of the [@openfloor/protocol](https://www.npmjs.com/package/@openfloor/protocol) package.

## What We're Building

Our chat demo will have two main features:

-   **Manifest Discovery**: Click a button to get an agent's capabilities
-   **Chat Interface**: Send utterance events and see responses in real-time

The best part? It's just one HTML file that you can open in any browser.

## Step 1: Setting Up the HTML Structure

Let's start by creating a new file called `openfloor-chat-demo.html`. We'll build this step by step, starting with the basic structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Open Floor Agent Chat Demo</title>
</head>
<body>
  <div class="container">
    <h1>Open Floor Agent Chat Demo</h1>
    
    <!-- Agent URL Input -->
    <label for="agentUrl">Agent URL:</label>
    <input type="text" id="agentUrl" value="https://your-agent-url.com/" />
    <button id="fetchManifestBtn">Get Manifest</button>
    
    <!-- Manifest Display -->
    <div class="manifest" id="manifestDisplay">
      Manifest will appear here.
    </div>
    
    <hr />
    
    <!-- Chat Interface -->
    <div class="chat-log" id="chatLog"></div>
    <div class="row">
      <input type="text" id="chatInput" placeholder="Type your message..." />
      <button id="sendBtn">Send</button>
    </div>
  </div>
</body>
</html>
```

**What we just created:**
-   A container for our entire chat interface
-   An input field for the agent URL
-   A button to fetch the agent's manifest
-   A display area for the manifest
-   A chat log to show the conversation
-   An input field and button for sending messages

## Step 2: Adding Some Style

Now let's make it look good with some clean, modern CSS. Add this inside the `<head>` section:

```html
<style>
  body { 
    font-family: Arial, sans-serif; 
    background: #f7f7f7; 
    margin: 0; 
    padding: 0; 
  }
  
  .container { 
    max-width: 600px; 
    margin: 40px auto; 
    background: #fff; 
    border-radius: 8px; 
    box-shadow: 0 2px 8px #0001; 
    padding: 24px; 
  }
  
  h1 { 
    font-size: 1.5em; 
    margin-bottom: 0.5em; 
  }
  
  label { 
    font-weight: bold; 
  }
  
  input[type="text"] { 
    width: 100%; 
    padding: 8px; 
    margin: 4px 0 12px 0; 
    border: 1px solid #ccc; 
    border-radius: 4px; 
  }
  
  button { 
    padding: 8px 16px; 
    border: none; 
    border-radius: 4px; 
    background: #0074d9; 
    color: #fff; 
    font-weight: bold; 
    cursor: pointer; 
  }
  
  button:disabled { 
    background: #aaa; 
  }
  
  .manifest, .chat-log { 
    background: #f4f4f4; 
    border-radius: 4px; 
    padding: 12px; 
    margin: 12px 0; 
    font-size: 0.95em; 
  }
  
  .chat-log { 
    height: 200px; 
    overflow-y: auto; 
    margin-bottom: 12px; 
  }
  
  .chat-msg { 
    margin: 6px 0; 
  }
  
  .chat-msg.user { 
    color: #0074d9; 
  }
  
  .chat-msg.agent { 
    color: #333; 
  }
  
  .row { 
    display: flex; 
    gap: 8px; 
  }
  
  .row input[type="text"] { 
    flex: 1; 
    margin: 0; 
  }
</style>
```

**What this styling does:**
-   Creates a clean, centered card layout
-   Styles inputs and buttons with a modern look
-   Sets up the chat log with proper scrolling
-   Colors user and agent messages differently
-   Makes the input row responsive

## Step 3: Adding the Open Floor Protocol Library

Now, we need to include the Open Floor JavaScript library. Add this script tag before the closing `</body>` tag:

```html
<script src="https://cdn.jsdelivr.net/npm/@openfloor/protocol@0.0.1/dist/bundle.js"></script>
```

**Why do we need this library?**
-   Provides helper functions for creating Open Floor messages
-   Handles the protocol structure automatically
-   Makes it easy to create envelopes, events, and payloads

## Step 4: Setting Up the JavaScript

Now that we have the basic HTML and CSS structure and also included the library, we can start adding the JavaScript that makes everything work. Add this after the script tag:

```html
<script>
  // Get references to our DOM elements
  const agentUrlInput = document.getElementById('agentUrl');
  const fetchManifestBtn = document.getElementById('fetchManifestBtn');
  const manifestDisplay = document.getElementById('manifestDisplay');
  const chatLog = document.getElementById('chatLog');
  const chatInput = document.getElementById('chatInput');
  const sendBtn = document.getElementById('sendBtn');

  // Initialize our variables
  let agentUrl = agentUrlInput.value;
  let speakerUri = 'user:' + Math.random().toString(36).slice(2, 10); // random user id

  // Update agent URL when input changes
  agentUrlInput.addEventListener('change', () => {
    agentUrl = agentUrlInput.value;
  });
</script>
```

**What we're setting up:**
-   References to all our HTML elements
-   A variable to track the current agent URL
-   A random speaker URI to identify our chat client

## Step 5: Implementing Manifest Fetching

Let's add the functionality to fetch and display agent manifests:

```javascript
fetchManifestBtn.addEventListener('click', async () => {
  manifestDisplay.textContent = 'Loading...';
  
  try {
    const conversationId = 'conv1';
    const senderUri = 'openfloor://localhost/TestClient';
    const senderServiceUrl = window.location.origin;

    // Build getManifests event and envelope
    const getManifestsEvent = new openfloor.GetManifestsEvent({
      to: { serviceUrl: agentUrl, private: false }
    });
    
    const envelope = openfloor.createSimpleEnvelope({
      conversationId,
      senderUri,
      senderServiceUrl,
      events: [getManifestsEvent]
    });
    
    const payload = envelope.toPayload();

    // POST the envelope to the agent
    const res = await fetch(agentUrl, {
      method: 'POST',
      headers: { 
        'Content-Type': 'application/json', 
        'Accept': 'application/json' 
      },
      body: JSON.stringify(payload.toObject())
    });
    
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const data = await res.json();

    // Try to extract manifest(s) from publishManifests event
    let manifests = [];
    try {
      const envelopeObj = data.openFloor;
      const publishEvent = envelopeObj.events.find(
        e => e.eventType === 'publishManifests' || e.eventType === 'publishManifest'
      );
      if (publishEvent && publishEvent.parameters && publishEvent.parameters.servicingManifests) {
        manifests = publishEvent.parameters.servicingManifests;
      }
    } catch (e) {
      manifests = [];
    }
    
    manifestDisplay.textContent = manifests.length
      ? JSON.stringify(manifests, null, 2)
      : '[No manifests found]';
      
  } catch (e) {
    manifestDisplay.textContent = 'Error fetching manifest: ' + e;
  }
});
```

**What this does:**
-   Creates a `getManifests` event using the Open Floor library
-   Wraps it in an envelope with proper conversation metadata
-   Sends it to the agent via HTTP POST
-   Parses the response to extract manifest information
-   Displays the manifest in a readable JSON format

## Step 6: Adding Chat Message Display

Let's create a helper function to display messages in our chat log:

```javascript
function appendMessage(text, who) {
  const div = document.createElement('div');
  div.className = 'chat-msg ' + who;
  div.textContent = (who === 'user' ? 'You: ' : 'Agent: ') + text;
  chatLog.appendChild(div);
  chatLog.scrollTop = chatLog.scrollHeight;
}
```

**What this helper does:**
-   Creates a new message element
-   Styles it based on who sent it (user or agent)
-   Adds it to the chat log
-   Automatically scrolls to show the latest message

## Step 7: Implementing Chat Functionality

Now, for the main chat functionality, you need to add the following:

```javascript
async function sendUtterance(text) {
  appendMessage(text, 'user');
  sendBtn.disabled = true;
  chatInput.disabled = true;
  
  try {
    // Use openfloor.createTextUtterance and openfloor.createSimpleEnvelope
    const utteranceEvent = openfloor.createTextUtterance({
      speakerUri,
      text
    });
    
    const envelope = openfloor.createSimpleEnvelope({
      conversationId: 'conv1',
      senderUri: speakerUri,
      events: [utteranceEvent]
    });
    
    const payload = envelope.toPayload();
    
    const res = await fetch(agentUrl, {
      method: 'POST',
      headers: { 
        'Content-Type': 'application/json', 
        'Accept': 'application/json' 
      },
      body: JSON.stringify(payload.toObject())
    });
    
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const data = await res.json();
    
    // Try to extract agent response from returned envelope
    let agentReply = '';
    try {
      const envelopeObj = openfloor.Payload.fromObject(data).openFloor;
      const utterance = envelopeObj.events.find(e => e.eventType === 'utterance');
      if (utterance && utterance.parameters && utterance.parameters.dialogEvent) {
        const features = utterance.parameters.dialogEvent.features;
        if (features && features.text && features.text.tokens && features.text.tokens.length > 0) {
          agentReply = features.text.tokens.map(t => t.value).join(' ');
        }
      }
    } catch (e) {
      agentReply = '[Could not parse agent reply]';
    }
    
    appendMessage(agentReply || '[No reply]', 'agent');
    
  } catch (e) {
    appendMessage('Error: ' + e, 'agent');
  } finally {
    sendBtn.disabled = false;
    chatInput.disabled = false;
    chatInput.focus();
  }
}
```

**What this function does:**
-   Shows the user's message immediately
-   Disables the input while processing
-   Creates an utterance event with the text
-   Sends it to the agent
-   Parses the response to extract the agent's reply
-   Displays the agent's response
-   Re-enables the input for the next message

## Step 8: Connecting the User Interface

Finally, let's connect our send button and enter key to the chat functionality:

```javascript
sendBtn.addEventListener('click', () => {
  const text = chatInput.value.trim();
  if (text) {
    chatInput.value = '';
    sendUtterance(text);
  }
});

chatInput.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    sendBtn.click();
  }
});
```

**What this does:**
-   Handles click events on the send button
-   Handles Enter key presses in the input field
-   Clears the input after sending
-   Only sends non-empty messages


## Using Your Chat Demo

1.  **Open the file** in your browser
2.  **Enter your agent URL** (like `http://localhost:8080/` for local development)
3.  **Click "Get Manifest"** to see what your agent can do
4.  **Type a message** and press Enter or click Send
5.  **Watch the conversation** unfold in the chat log

If these steps do not work check if there are access issues due to a CORS configuration on the agent side. 

## Testing with Your Parrot Agent

If you've built the [parrot agent from our previous tutorial](https://openfloor.dev/tutorials/building-a-parrot-agent), you can test it by:

1.  Starting your parrot agent server
2.  Using `http://localhost:8080/` as the agent URL
3.  Sending messages like "Hello!" and watching it echo back "🦜 Hello!"

---

If you found this guide useful, try building different types of agents and see how they behave in the chat interface.