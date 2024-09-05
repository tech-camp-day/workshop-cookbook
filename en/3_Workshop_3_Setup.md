# Workshop 2: Creating a Smart Bot

In this workshop, we will take on the role of store owners and create two bots: one for members to check their points, and another for the store to manage its members.

## Step 0: Copy the Project
1. Go to https://codesandbox.io/p/devbox/e-membership-starter-zxdpgs
2. Click the "Fork" button in the top right corner of the page

After forking, the project on your screen will become your own, and you can edit it as you like.

3. Click the "Share" button in the top right corner of the page
4. Click "Move out of Drafts"
5. In "Change permissions," select "Unlisted" to make the project accessible
6. Click the square box in the top left corner and select "Restart Devbox"

## Step 0.5: Brief Explanation of the Code (Skip if you're already familiar)

The project you just copied contains 4 important files:
- `index.js` - The main file of the project; the program starts here
- `persists.js` - Contains various functions related to database interactions
- `member.js` - Code for the member bot
- `merchant.js` - Code for the store bot

Additionally, there are functions for sending messages:
- `pushToMember` - For sending `push` messages to members
- `replyToMember` - For sending reply messages to members
- `replyToMerchant` - For sending reply messages to the store

## Step 1: Set Up the Member Bot

1. Create a new bot following the steps in [this file](0_Create_LINE_bot.md)
2. After creating the bot, go to the "Basic settings" page
3. Copy the Channel secret and replace `MEMBER_CHANNEL_SECRET` in `member.js` with it
4. Go to the "Messaging API" page
5. Copy the Channel access token and replace `MEMBER_CHANNEL_ACCESS_TOKEN` in the same file
6. In Codesandbox, click Save (`Ctrl` + `S`); the program will restart automatically
7. In Codesandbox, the right half of the screen is a Browser with a link at the top. Copy that link and paste it into the "Messaging API" page of the bot in the "Webhook URL" field, appending `/webhooks/member` 
> Example: `https://www.your-url.com/webhooks/member`
> ![image](https://github.com/tech-camp-day/workshop-cookbook/assets/47054457/2d70f3b5-6900-4c88-a892-0c107df5701f)

8. Click "Update" and then "Verify" to test the connection between LINE and the bot. If successful, "Success" will appear
9. Enable "Use webhook" if it’s not already enabled

## Step 2: Add Members to the System When They Add the Bot as a Friend

1. In `member.js`, create a `handleFollowEvent` function before the `handleUnknownEvent` function to handle the `follow` event we receive when someone adds our bot as a friend

```js
function handleFollowEvent(event) {
    const userId = event.source.userId;

    insertMember(userId);
    pushToMember(userId, "Welcome to the Membership Bot! Please type 'Check Points' to check your points.");
}
```

---
__Explanation__

To add a member to the system, we need the `userId` which we can obtain from `event.source.userId`

```js
const userId = event.source.userId;
```

Once we have the `userId`, we add the member to the system using the existing `insertMember` function

```js
insertMember(userId);
```

Then, we send a welcome message to the member using the `pushToMember` function

```js
pushToMember(userId, "Welcome to the Membership Bot! Please type 'Check Points' to check your points.");
```

---

2. In the `handleLineEvent` function, call the `handleFollowEvent` function you just created when `event.type` is `follow`. You can add this code to the `switch` statement

```js
case "follow":
    return handleFollowEvent(event);
```

Now, the `handleLineEvent` function should look like this:

```js
function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}
```

## Step 3: Remove Members from the System When They Block the Bot

1. In `member.js`, create a `handleUnfollowEvent` function before the `handleUnknownEvent` function to handle the `unfollow` event we receive when someone blocks us or removes us from their friends

```js
function handleUnfollowEvent(event) {
  const userId = event.source.userId;

  deleteMember(userId);
}
```

---
__Explanation__

To remove a member from the system, we need the `userId` which we can obtain from `event.source.userId`

```js
const userId = event.source.userId;
```

Once we have the `userId`, we remove the member from the system using the existing `deleteMember` function

```js
deleteMember(userId);
```

---

2. In the `handleLineEvent` function, call the `handleUnfollowEvent` function you just created when `event.type` is `unfollow`. You can add this code to the `switch` statement

```js
case "unfollow":
    return handleUnfollowEvent(event);
```

Now, the `handleLineEvent` function should look like this:

```js
function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    case "unfollow":
      return handleUnfollowEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}
```

## Step 4: Allow Members to Check Their Points

> When a member sends the message `Check Points`, respond with the message `You have XXX points`

1. In `member.js`, create a `handleMessageEvent` function before the `handleUnknownEvent` function to handle the `message` event we receive when someone sends us a message

```js
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "Check Points") {
    const member = getMemberByUid(event.source.userId);
    return replyToMember(event, `You have ${member.points} points.`);
  }
}
```

---
__Explanation__

To check if the member sent the message `Check Points`, we use the `text` from `event.message.text`

```js
const text = event.message.text;
```

After getting the text, we check if it matches `Check Points`

```js
if (text === "Check Points") {
    // some other code
}
```

To find the member, we need the `userId` which we get from `event.source.userId`

```js
const userId = event.source.userId;
```

With the `userId`, we find the member using the existing `getMemberByUid` function

```js
const member = getMemberByUid(userId);
```

Then, we send the points information back to the member using the `replyToMember` function

```js
return replyToMember(event, `You have ${member.points} points.`);
```

---

2. In the `handleLineEvent` function, call the `handleMessageEvent` function you just created when `event.type` is `message`. You can add this code to the `switch` statement

```js
case "message":
    return handleMessageEvent(event);
```

Now, the `handleLineEvent` function should look like this:

```js
function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    case "unfollow":
      return handleUnfollowEvent(event);
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}
```

3. Try sending a message to the bot

## Step 4.5: Member Cheat Code

For those who are having trouble, copy this code and replace the code in `member.js`, change `MEMBER_CHANNEL_SECRET` and `MEMBER_CHANNEL_ACCESS_TOKEN`, then click Save and test adding, blocking, and unblocking friends

<details>
<summary>Full code for <code>member.js</code></summary>

```js
const express = require("express");
const line = require("@line/bot-sdk");
const { insertMember, deleteMember, getMemberByUid } = require("../persists");
const { replier, pusher } = require("./util");

const memberConfig = {
  channelSecret: "MEMBER_CHANNEL_SECRET",
  channelAccessToken:
    "MEMBER_CHANNEL_ACCESS_TOKEN",
};

const memberWebhookMiddleware = line.middleware(memberConfig);
const memberClient = new line.messagingApi.MessagingApiClient(memberConfig);
const replyToMember = replier(memberClient);
const pushToMember = pusher(memberClient);

const router = express.Router();

router.post("/", memberWebhookMiddleware, handlePostRequest);

/**
 * Handle requests to /webhooks/member
 * @param {Object} request - Incoming request
 * @param {Object} response - Response to the request
 * @returns {Promise} - Returns the result of event handling
 */
async function handlePostRequest(request, response) {
  // Extract events from request body
  const { events } = request.body;

  // Handle each event


  await Promise.all(events.map(handleLineEvent));

  // Send a 200 OK response
  response.sendStatus(200);
}

/**
 * Handle LINE events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns the result of event handling
 */
function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    case "unfollow":
      return handleUnfollowEvent(event);
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}

/**
 * Handle follow events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns the result of follow event handling
 */
function handleFollowEvent(event) {
  const userId = event.source.userId;

  insertMember(userId);
  pushToMember(userId, "Welcome to the Membership Bot! Please type 'Check Points' to check your points.");
}

/**
 * Handle unfollow events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns the result of unfollow event handling
 */
function handleUnfollowEvent(event) {
  const userId = event.source.userId;

  deleteMember(userId);
}

/**
 * Handle message events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns the result of message event handling
 */
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "Check Points") {
    const member = getMemberByUid(event.source.userId);
    return replyToMember(event, `You have ${member.points} points.`);
  }
}

/**
 * Handle unknown events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns an empty result
 */
function handleUnknownEvent(event) {
  // Do nothing for unknown events
}

module.exports = router;
```
</details>

## Step 5: Set Up the Store Bot

1. Create a new bot following the steps in [this file](0_Create_LINE_bot.md)
2. After creating the bot, go to the "Basic settings" page
3. Copy the Channel secret and replace `MERCHANT_CHANNEL_SECRET` in `merchant.js` with it
4. Go to the "Messaging API" page
5. Copy the Channel access token and replace `MERCHANT_CHANNEL_ACCESS_TOKEN` in the same file
6. In Codesandbox, click Save (`Ctrl` + `S`); the program will restart automatically
7. In Codesandbox, the right half of the screen is a Browser with a link at the top. Copy that link and paste it into the "Messaging API" page of the bot in the "Webhook URL" field, appending `/webhooks/merchant` 
> Example: `https://www.your-url.com/webhooks/merchant`
> ![image](https://github.com/tech-camp-day/workshop-cookbook/assets/47054457/2d70f3b5-6900-4c88-a892-0c107df5701f)

8. Click "Update" and then "Verify" to test the connection between LINE and the bot. If successful, "Success" will appear
9. Enable "Use webhook" if it’s not already enabled

## Step 6: Add Commands for the Store Bot

1. In `merchant.js`, create a `handleMessageEvent` function before the `handleUnknownEvent` function to handle the `message` event we receive when someone sends us a message

```js
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "Add Points") {
    const userId = event.source.userId;
    const member = getMemberByUid(userId);

    if (member) {
      addPoints(member.userId, 100);
      return replyToMerchant(event, "Added 100 points to your account.");
    } else {
      return replyToMerchant(event, "You are not a member.");
    }
  }

  if (text === "Check All Members") {
    const members = getAllMembers();
    let responseText = "Members:\n";

    members.forEach(member => {
      responseText += `UserID: ${member.userId}, Points: ${member.points}\n`;
    });

    return replyToMerchant(event, responseText);
  }
}
```

---
__Explanation__

For the `Add Points` command:
- Get the `userId` from `event.source.userId` and use it to find the member using `getMemberByUid`
- If the member exists, use the existing `addPoints` function to add 100 points to their account and reply with a success message
- If the member does not exist, reply with a "not a member" message

For the `Check All Members` command:
- Use the existing `getAllMembers` function to retrieve all members
- Format the response text with each member's `userId` and `points` and reply with this information

---

2. In the `handleLineEvent` function, call the `handleMessageEvent` function you just created when `event.type` is `message`. You can add this code to the `switch` statement

```js
case "message":
    return handleMessageEvent(event);
```

Now, the `handleLineEvent` function should look like this:

```js
function handleLineEvent(event) {
  switch (event.type) {
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}
```

3. Try sending the `Add Points` and `Check All Members` commands to the bot

## Step 6.5: Store Bot Cheat Code

For those who are having trouble, copy this code and replace the code in `merchant.js`, change `MERCHANT_CHANNEL_SECRET` and `MERCHANT_CHANNEL_ACCESS_TOKEN`, then click Save and test the `Add Points` and `Check All Members` commands

<details>
<summary>Full code for <code>merchant.js</code></summary>

```js
const express = require("express");
const line = require("@line/bot-sdk");
const { getMemberByUid, addPoints, getAllMembers } = require("../persists");
const { replier } = require("./util");

const merchantConfig = {
  channelSecret: "MERCHANT_CHANNEL_SECRET",
  channelAccessToken:
    "MERCHANT_CHANNEL_ACCESS_TOKEN",
};

const merchantWebhookMiddleware = line.middleware(merchantConfig);
const merchantClient = new line.messagingApi.MessagingApiClient(merchantConfig);
const replyToMerchant = replier(merchantClient);

const router = express.Router();

router.post("/", merchantWebhookMiddleware, handlePostRequest);

/**
 * Handle requests to /webhooks/merchant
 * @param {Object} request - Incoming request
 * @param {Object} response - Response to the request
 * @returns {Promise} - Returns the result of event handling
 */
async function handlePostRequest(request, response) {
  // Extract events from request body
  const { events } = request.body;

  // Handle each event
  await Promise.all(events.map(handleLineEvent));

  // Send a 200 OK response
  response.sendStatus(200);
}

/**
 * Handle LINE events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns the result of event handling
 */
function handleLineEvent(event) {
  switch (event.type) {
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}

/**
 * Handle message events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns the result of message event handling
 */
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "Add Points") {
    const userId = event.source.userId;
    const member = getMemberByUid(userId);

    if (member) {
      addPoints(member.userId, 100);
      return replyToMerchant(event, "Added 100 points to your account.");
    } else {
      return replyToMerchant(event, "You are not a member.");
    }
  }

  if (text === "Check All Members") {
    const members = getAllMembers();
    let responseText = "Members:\n";

    members.forEach(member => {
      responseText += `UserID: ${member.userId}, Points: ${member.points}\n`;
    });

    return replyToMerchant(event, responseText);
  }
}

/**
 * Handle unknown events
 * @param {Object} event - LINE event
 * @returns {Promise} - Returns an empty result
 */
function handleUnknownEvent(event) {
  // Do nothing for unknown events
}

module.exports = router;
```
</details>
