##### initial streaming function
```js
let chunks = [];
for await (let chunk of response.body) {
  chunks.push(chunk);
}

let chunk = Buffer.concat(chunks).toString(); // Convert Buffer to string

// console.log(chunk);
const lines = chunk.split('\n');
const parsedLines = lines
  .map((line) => line.replace(/^data: /, '').trim()) // Remove the "data: " prefix
  .filter((line) => line !== '' && line !== '[DONE]') // Remove empty lines and "[DONE]"
  .map((line) => JSON.parse(line)); // Parse the JSON string

// const responseId = Date.now();

  for (const parsedLine of parsedLines) {
    const { choices } = parsedLine;
    const { delta } = choices[0];
    const { content } = delta;
    if (content) {
      const botResponse = content;
      // console.log(`Content: ${botResponse}`);
      // assistantContent += botResponse;
      sendSse(responseId, { botResponse });
    }
  }
messages.push({
  role: 'assistant',
  content: `${assistantContent}`,
});
const conversation = messages;
// console.log(messages);

sendSse(responseId, { conversation });
```

##### check if the server is live
```js

app.get('/', async (req, res) => {
  res.status(200).send({
    message: 'Hello from Codex',
  });
});
```

It seems like `model` is not being properly passed in the query string parameters. Based on the code snippet you provided, here's a walkthrough of what may be causing the issue:

1. In the `EventSource` URL, you are trying to encode the model object using `JSON.stringify(model)`. Ensure that `model` is properly defined and has a value before encoding it.

2. When parsing the query parameters on Express.js using `req.query.model`, make sure you have the necessary middleware to parse the URL-encoded request query parameters.

Here's what you can do to debug this issue:

1. Check if the `model` has a value before encoding it in the `EventSource`. You can do this by logging the value of `model`:

   ```javascript
   console.log("model before encoding:", model);
   ```

   If it is undefined, you may need to look at where `model` is defined and ensure it's getting a value.

2. Make sure you are using middleware to parse the URL-encoded request query parameters in your Express application. Including the `express.urlencoded` middleware should help:

   ```javascript
   const express = require('express');
   const app = express();

   app.use(express.urlencoded({ extended: true }));
   ```

   ```js
    const response = await openai.createChatCompletion({
    // model: 'gpt-4',
      model: 'gpt-3.5-turbo',
      messages: conversation,
      temperature: 0.7,
      max_tokens: 500,
      stream: true,
    });
    ```