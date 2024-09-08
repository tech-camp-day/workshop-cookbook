# Workshop 1: Creating a Simple Bot

In this workshop, we will create a fortuneteller bot by randomly selecting predictions from a predefined set of data.

## Step 0: Copy the Project

1. Go to [this link](https://codesandbox.io/p/devbox/musing-goldberg-nz2dc4).
2. Click the "Fork" button in the top-right corner of the page.

After forking, the project you see on the screen will become your own, and you can modify it as needed.

3. Click the "Share" button in the top-right corner of the page.
4. Click "Move out of Drafts."
5. In "Change permissions," select "Unlisted" to make the project accessible.
6. Click the square box in the top-left corner, then click "Restart Devbox."

## Step 1: Configure the Bot

1. Create a new bot following the steps in [this file](0_Create_LINE_bot.md).
2. After creating the bot, go to the bot's "Basic settings" page.
3. Copy the Channel secret and replace `CHANNEL_SECRET` in the `index.js` file.
4. Go to the bot's "Messaging API" page.
5. Copy the Channel access token and replace `CHANNEL_ACCESS_TOKEN` in the same file.
6. In Codesandbox, click Save (`Ctrl` + `S`), and the program will automatically restart.

## Step 1.5: Brief Explanation of the Code (Skip if you’re already familiar)

The project you just copied includes two important files:
- `index.js`: The main file of the project where we will write the bot's code. The program starts from this file.
- `fortune.json`: Contains prediction data to be sent back to users.

## Step 2: Enable "Fortune-Telling Bot" to Reply to Messages

1. In `index.js`, add the following code under the `lineConfig` variable:

```javascript
const messagingClient = new line.messagingApi.MessagingApiClient(lineConfig);
```

The `messagingClient` created here helps in using LINE to send messages back to users.

2. Add the following code at the end of the `index.js` file:

```javascript
function reply(event, ...messages) {
  return messagingClient.replyMessage({
    replyToken: event.replyToken,
    messages: messages.map(message => ({
      type: "text",
      text: message,
    })),
  });
}
```

The `reply` function uses the `messagingClient` to make it easier to send messages back to users.

3. Add the following code at the end of the `index.js` file, after the code you just added:

```javascript
function handleLineEvent(event) {
  return reply(event, "Hello from Fortune-Telling Bot");
}
```

4. Click Save.
5. In Codesandbox, the right side of the screen will show a Browser with a link at the top. Copy that link and paste it into the "Webhook URL" field on the bot's "Messaging API" page.
6. Click "Update" and then "Verify" to test the connection between LINE and the bot; if successful, "Success" will appear.
7. Click to enable "Use webhook" if it isn't already enabled.
8. Config your bot following [this file](0_2_Config_LINE_bot.md)
9. Try sending any message in LINE.

## Step 3: Randomly Generate Fortune

1. Create constants for each type of fortune for convenience:

```js
const FortuneCategory = {
  Love: "love",
  Career: "career",
  Finance: "finance",
  Health: "health",
};
```

2. Create a `getFortune` function for random fortune in each category:

```js
function getFortune(category) {
  const index = Math.floor(Math.random() * fortunes[category].length);
  return fortunes[category][index];
}
```

3. Create a `counter` variable to keep track of how many times the fortune-telling has been done:

```js
let count = 1;
```

4. Create a `getAllFortuneMessages` function to randomly generate one fortune for each category and format it as a LINE message:

```js
function getAllFortuneMessages() {
  const fortuneMessages = [
    `Daily fortune-telling round ${count++}`,
    `Love\n${getFortune(FortuneCategory.Love)}`,
    `Career\n${getFortune(FortuneCategory.Career)}`,
    `Finance\n${getFortune(FortuneCategory.Finance)}`,
    `Health\n${getFortune(FortuneCategory.Health)}`,
  ];

  return fortuneMessages;
}
```

5. Modify the `handleLineEvent` function to respond with the newly generated fortune:

```js
function handleLineEvent(event) {
  return reply(event, ...getAllFortuneMessages());
}
```

6. Click Save and try sending any message in LINE.

## Step 4: Respond Only to the Keyword "tell"

At this point, the bot responds with fortunes every time a message is sent. We want the bot to respond only when the message is "tell"

1. Modify the `handleLineEvent` function to check if the incoming message is "tell":

```js
function handleLineEvent(event) {
  // Extract text from the message in the event
  const text = event.message.text;

  if (text === "tell") {
    return reply(event, ...getAllFortuneMessages());
  }
}
```

2. Click Save and try sending any message in LINE; if it's not "tell", the bot will not respond.

3. Modify the `handleLineEvent` function again to send a different message if the incoming text is not "tell":

```js
function handleLineEvent(event) {
  // Extract text from the message in the event
  const text = event.message.text;

  if (text === "tell") {
    return reply(event, ...getAllFortuneMessages());
  } else {
    return reply(event, "I don't understand. Type 'tell' to get a fortune.");
  }
}
```

4. Click Save and try sending various messages in LINE.

## Step 99: Cheat Code

For those who are struggling, you can replace the code in `index.js` with the code below, change `CHANNEL_SECRET` and `CHANNEL_ACCESS_TOKEN`, click Save, and try sending any message in LINE.

<details>
<summary>Full index.js Code</summary>

```javascript
const line = require("@line/bot-sdk");
const express = require("express");
const fortunes = require("./fortune.json");

const lineConfig = {
  channelSecret: "CHANNEL_SECRET",
  channelAccessToken: "CHANNEL_ACCESS_TOKEN",
};
const messagingClient = new line.messagingApi.MessagingApiClient(lineConfig);

const app = express();
app.post("/", line.middleware(lineConfig), handlePostRequest);
app.listen(3000);

async function handlePostRequest(req, res) {
  const { events } = req.body;

  const eventHandledPromises = events.map(handleLineEvent);

  const result = await Promise.all(eventHandledPromises);

  return res.send(result);
}

function reply(event, ...messages) {
  return messagingClient.replyMessage({
    replyToken: event.replyToken,
    messages: messages.map((message) => ({
      type: "text",
      text: message,
    })),
  });
}

function handleLineEvent(event) {
  const text = event.message.text;

  if (text === "tell") {
    return reply(event, ...getAllFortuneMessages());
  } else {
    return reply(event, "I don't understand. Type 'tell' to get a fortune.");
  }
}

const FortuneCategory = {
  Love: "love",
  Career: "career",
  Finance: "finance",
  Health: "health",
};

function getFortune(category) {
  const index = Math.floor(Math.random() * fortunes[category].length);
  return fortunes[category][index];
}

let count = 1;

function getAllFortuneMessages() {
  const fortuneMessages = [
    `Daily fortune-telling round ${count++}`,
    `Love\n${getFortune(FortuneCategory.Love)}`,
    `Career\n${getFortune(FortuneCategory.Career)}`,
    `Finance\n${getFortune(FortuneCategory.Finance)}`,
    `Health\n${getFortune(FortuneCategory.Health)}`,
  ];

  return fortuneMessages;
}
```

</details>
