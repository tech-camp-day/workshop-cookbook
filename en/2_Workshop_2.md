# Workshop 2: Creating a Smarter Bot

In this workshop, we'll take on the role of a store owner, creating one bot for members to check their points, and another bot for store management of members.

## Step 0: Copy the Project
1. Go to [this link](https://codesandbox.io/p/devbox/e-membership-template-en-mwymkr).
2. Click the "Fork" button in the top right corner of the page.

After forking, the project displayed on the screen will become your own, and you can edit it as needed.

3. Click the "Share" button in the top right corner of the page.
4. In "Change permissions," select "Unlisted" to make the project accessible.
5. Click the square box in the top left corner and then click "Restart Devbox."

## Step 0.5: Brief Code Explanation (Skip if you're already proficient)

The project you copied contains four important files:
- `index.js`: The main file of the project; the program starts here.
- `persists.js`: Contains functions related to interacting with the database.
- `member.js`: Code for the member bot.
- `merchant.js`: Code for the store bot.

Additionally, there are functions for sending messages:
- `pushToMember`: For sending push messages to members.
- `replyToMember`: For replying to members.
- `replyToMerchant`: For replying to the store.

## Step 1: Set Up the Member Bot

1. Create a new bot following the steps in [this file](0_Create_LINE_bot.md).
2. After creating the bot, go to the "Basic settings" page of the bot.
3. Copy the Channel secret and replace `MEMBER_CHANNEL_SECRET` in the `member.js` file.
4. Go to the "Messaging API" page of the bot.
5. Copy the Channel access token and replace `MEMBER_CHANNEL_ACCESS_TOKEN` in the same file.
6. In Codesandbox, click Save (`Ctrl` + `S`), and the program will automatically restart.
7. In Codesandbox, the right half of the screen will be the browser with a link at the top. Copy that link and paste it into the "Messaging API" page of the bot in the "Webhook URL" field, appending `/webhooks/member` to the end.
> Example `https://www.your-url.com/webhooks/member`
> ![image](https://github.com/tech-camp-day/workshop-cookbook/assets/47054457/2d70f3b5-6900-4c88-a892-0c107df5701f)

8. Click "Update" and then "Verify" to test the connection between LINE and the bot; if successful, it will show "Success."
9. Enable "Use webhook" if it's not already turned on.
10. Make sure you configure your bot(or channel) following [this file](0_2_Config_LINE_bot.md)

## Step 2: Add Members to the System When They Follow the Bot

1. In `member.js`, create a function `handleFollowEvent` before `handleUnknownEvent` to handle the `follow` event when someone adds the bot as a friend.

```js
function handleFollowEvent(event) {
  const userId = event.source.userId;

  const { id: memberId } = insertMember(userId);
  pushToMember(
    userId,
    "Welcome to the club. Please type 'check' to check your score.",
    `Your member number is ${memberId}`
  );
}
```

---
__Explanation__

To add a member to the system, use the `userId` which you can get from `event.source.userId`.

```js
const userId = event.source.userId;
```

Once you have the `userId`, add the member to the system using the existing `insertMember` function.

```js
insertMember(userId);
```

Then send a welcome message to the member using `pushToMember`.

```js
  pushToMember(
    userId,
    "Welcome to the club. Please type 'check' to check your score.",
    `Your member number is ${memberId}`
  );
```

---

2. In `handleLineEvent`, call the `handleFollowEvent` function when `event.type` is `follow`. You can add this code to the `switch` statement.

```js
case "follow":
    return handleFollowEvent(event);
```

The `handleLineEvent` function will now look like this:

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

1. In `member.js`, create a function `handleUnfollowEvent` before `handleUnknownEvent` to handle the `unfollow` event when someone blocks or removes the bot from their friends.

```js
function handleUnfollowEvent(event) {
  const userId = event.source.userId;

  deleteMember(userId);
}
```

---
__Explanation__

To remove a member from the system, use the `userId` which you can get from `event.source.userId`.

```js
const userId = event.source.userId;
```

Once you have the `userId`, remove the member from the system using the existing `deleteMember` function.

```js
deleteMember(userId);
```

---

2. In `handleLineEvent`, call the `handleUnfollowEvent` function when `event.type` is `unfollow`. Add this code to the `switch` statement.

```js
case "unfollow":
    return handleUnfollowEvent(event);
```

The `handleLineEvent` function will now look like this:

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

## Step 3.1 Register by Blocking/Unblocking

1. Since the `handleFollowEvent` function only triggers when a friend is added, we need to block the bot first, simulating an unfollow.
2. Go to your LINE Bot and select the three-line menu.
3. Select block.
4. Select unblock to re-add the bot. The bot will greet you again.


## Step 4: Allow Members to Check Their Points

> When a member sends the message `Check Points`, reply with the message `You have XXX points`.

1. In `member.js`, create a function `handleMessageEvent` before `handleUnknownEvent` to handle `message` events when someone sends a message.

```js
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "check") {
    const userId = event.source.userId;
    const member = getMemberByUid(userId);

    if (!member) {
      return replyToMember(
        event,
        "Sorry, you are not a member of the store yet. Please block and unblock the bot to start over."
      );
    }
    return replyToMember(event, `You have ${member.points} points.`);
  } else {
    return replyToMember(event, "I don't understand the command.");
  }
}
```

---
__Explanation__

To check if the member sent the message `Check Points`, use the message `text` from `event.message.text`.

```js
const text = event.message.text;
```

Once you have the text, check if it matches `Check Points`.

```js
if (text === "check") {
    // some other code
} else {
    // unknown command
} 
```

To find the member, use the `userId` which you can get from `event.source.userId`.

```js
const userId = event.source.userId;
```

With the `userId`, retrieve the member using the existing `getMemberByUid` function.

```js
const member = getMemberByUid(userId);
```

Then, send the points information back to the member using `replyToMember`.

```js
return replyToMember(event, `You have ${member.points} points.`);
```

---

2. In `handleLineEvent`, call the `handleMessageEvent` function when `event.type` is `message`. Add this code to the `switch` statement.

```js
case "message":
    return handleMessageEvent(event);
```

The `handleLineEvent` function will now look like this:

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

3. Test by sending a message to the bot.

## Step 4.5: Member Cheat Code

For those who need a shortcut, copy this code to replace the code in `member.js`, replace `MEMBER_CHANNEL_SECRET` and `MEMBER_CHANNEL_ACCESS_TOKEN`, and click Save. Then test adding, blocking, and unblocking friends.

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
 * Handles requests to /webhooks/member
 * @param {Object} request - Incoming request
 * @param {Object} response - Response to the request
 * @returns {Promise} - Result of event handling
 */
async function handlePostRequest(request, response) {
  // Extract events from request body
  const { events } = request.body;

  // Call handleLineEvent for each event
  const eventHandledPromises = events.map(handleLineEvent);

  // Wait for all events to be handled
  const result = await Promise.all(eventHandledPromises);

  // Send result of event handling
  return response.send(result);
}

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

function handleFollowEvent(event) {
  const userId = event.source.userId;

  const { id: memberId } = insertMember(userId);
  pushToMember(
    userId,
    "Welcome to the club. Please type 'check' to check your score.",
    `Your member number is ${memberId}`
  );
}

function handleUnfollowEvent(event) {
  const userId = event.source.userId;

  deleteMember(userId);
}

function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "check") {
    const userId = event.source.userId;
    const member = getMemberByUid(userId);

    if (!member) {
      return replyToMember(
        event,
        "Sorry, you are not a member of the store yet. Please block and unblock the bot to start over."
      );
    }
    return replyToMember(event, `You have ${member.points} points.`);
  } else {
    return replyToMember(event, "I don't understand the command.");
  }
}

function handleUnknownEvent(event) {
  return replyToMember(event, "Sorry, I don't understand your command");
}

module.exports = {
  router,
  pushToMember,
};

```

</details>

## Step 5: Configure the First Bot - Merchant Side Bot

1. Create a new bot following the steps in [this file](0_Create_LINE_bot.md).
2. After creating the bot, go to the bot's "Basic settings" page.
3. Copy the Channel Secret and replace `MERCHANT_CHANNEL_SECRET` in the `merchant.js` file.
4. Go to the bot's "Messaging API" page.
5. Copy the Channel Access Token and replace `MERCHANT_CHANNEL_ACCESS_TOKEN` in the same file.
6. Scroll down to "LINE Official Account features," find "Auto-reply messages," click "Edit," and disable all features except "Webhook."
7. In Codesandbox, click Save (`Ctrl` + `S`). The program will automatically restart.
8. In Codesandbox, the right half of the screen is a browser with a link at the top. Copy that link and paste it into the "Webhook URL" field on the bot's "Messaging API" page, appending `/webhooks/merchant` to it.
   > Example: `https://www.your-url.com/webhooks/merchant`
9. Click "Update" and then "Verify" to test the connection between LINE and the bot. If successful, you will see the message "Success."
10. Turn on "Use webhook" if it is not already enabled, and then try sending any message in LINE.

## Step 5.5: Brief Explanation of the Code (Skip if You're Already Familiar)

In `merchant.js`, we already have some code, such as:

1. The `handleLineEvent` function to manage events from LINE.
2. The `handleMessageEvent` function to handle `message` type events, including member management and handling messages that are not predefined.

## Step 5: Allow the Store to Add Points for Members

1. In `merchant.js`, create the `handlePointIncrease` function before the `handleUnknownEvent` function to handle adding points to members.

```js
function handlePointIncrease(event, text) {
  const [_, memberId, pointToAddText] = text.split(" ");

  if (!memberId || !pointToAddText)
    return replyToMerchant(event, "Invalid command. Add points with the command 'add <member ID> <points>'");

  const member = getMemberById(memberId);

  if (!member) {
    return replyToMerchant(event, `Member with ID ${memberId} not found`);
  }

  const pointToAdd = parseInt(pointToAddText);

  if (isNaN(pointToAdd)) {
    return replyToMerchant(event, "Please enter the number of points to add as a number only");
  }

  updatePointForMember(memberId, member.points + pointToAdd);
  pushToMember(member.uid, `You have received ${pointToAdd} points`);

  return replyToMerchant(event, `Added ${pointToAdd} points to member with ID ${memberId}`);
}
```

---
__Explanation__

Since the command to add points to a member will come in the format `add <member ID> <points to add>`, we need to extract the member ID and the points to add using:

```js
const [_, memberId, pointToAddText] = text.split(" ");
```

To add points, we need to take the current points and add the new points to it: `new points = current points + points to add`. Therefore, we need to find the member to update the points using the `getMemberById` function:

```js
const member = getMemberById(memberId);
```

If the member is not found, we should not add the points and instead send a message to the merchant saying `Member ID XXXX not found` using the `replyToMerchant` function:

```js
if (!member) {
    return replyToMerchant(event, `Member ID ${memberId} not found`);
}
```

The points received from the merchant's command will still be in text format, so we need to convert it to a number using `parseInt`. If the value is not a number, send a message back to the merchant saying `Please enter a numeric value for the points to add`:

```js
const pointToAdd = parseInt(pointToAddText);

if (isNaN(pointToAdd)) {
    return replyToMerchant(event, "Please enter a numeric value for the points to add");
}
```

Once we have both the member ID and the new points, we can use the `updatePointForMember` function to update the member's points:

```js
updatePointForMember(memberId, member.points + pointToAdd);
```

After that, we can send a notification to the member using the `pushToMember` function:

```js
pushToMember(member.uid, `You have received ${pointToAdd} points`);
```

Finally, send a confirmation message back to the merchant saying `Added XXXX points to member ID YYY`:

```js
return replyToMerchant(event, `Added ${pointToAdd} points to member ID ${memberId}`);
```

---

2. In the `handleMessageEvent` function, add a condition to check if the message contains "Add points" (add points) and call the `handlePointIncrease` function created earlier:

```js
if (text.includes("add")) {
    return handlePointIncrease(event, text);
}
```

The `handleMessageEvent` function will look something like this:

```js
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "manage") {
    const members = getAllMembers();

    if (!members || members.length === 0) 
      return replyToMerchant(event, "You currently have no members");
    const memberInfoMessages = members.map((member) => `Member number ${member.id} has ${member.points} points`);

    return replyToMerchant(event, ...memberInfoMessages);
  }

  if (text.includes("add")) {
    return handlePointIncrease(event, text);
  }

  return replyToMerchant(
    event,
    "Please type 'manage' to view member information or to add points to a member",
    "or type 'add <Member ID> <Points>' to add points to a member",
  );
}
```

## Step 5.5: Merchant Cheat Code

For those who are struggling, copy this code and replace the code in the `merchant.js` file, update `MERCHANT_CHANNEL_SECRET` and `MERCHANT_CHANNEL_ACCESS_TOKEN`, then click Save and try it out.

<details>
<summary>Full code for <code>merchant.js</code></summary>

```js
const express = require("express");
const line = require("@line/bot-sdk");
const {
  getMemberById,
  getAllMembers,
  updatePointForMember,
  getMemberByUid,
} = require("../persists");
const { replier } = require("./util");
const {
  pushToMember,
} = require("./member");

const merchantConfig = {
  channelSecret: "MERCHANT_CHANNEL_SECRET",
  channelAccessToken:
    "MERCHANT_ACCESS_TOKEN",
};

const merchantWebhookMiddleware = line.middleware(merchantConfig);
const merchantClient = new line.messagingApi.MessagingApiClient(merchantConfig);
const replyToMerchant = replier(merchantClient);

const router = express.Router();

router.post("/", merchantWebhookMiddleware, handleRequest);

/**
 * Handle requests coming to /webhooks/member
 * @param {Object} request - Incoming request
 * @param {Object} response - Response to the request
 * @returns {Promise} - Result of event handling
 */
async function handleRequest(request, response) {
  // Extract events from request body
  const { events } = request.body;

  // Call handleLineEvent function for each event
  const eventHandledPromises = events.map(handleLineEvent);

  // Wait for all events to be handled
  const result = await Promise.all(eventHandledPromises);

  // Send the result of event handling back
  return response.send(result);
}

function handleLineEvent(event) {
  switch (event.type) {
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}

function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "manage") {
    const members = getAllMembers();

    if (!members || members.length === 0) 
      return replyToMerchant(event, "You currently have no members");
    const memberInfoMessages = members.map((member) => `Member number ${member.id} has ${member.points} points`);

    return replyToMerchant(event, ...memberInfoMessages);
  }

  if (text.includes("add")) {
    return handlePointIncrease(event, text);
  }

  return replyToMerchant(
    event,
    "Please type 'manage' to view member information or to add points to a member",
    "or type 'add <Member ID> <Points>' to add points to a member",
  );
}

function handlePointIncrease(event, text) {
  const [_, memberId, pointToAddText] = text.split(" ");

  if (!memberId || !pointToAddText)
    return replyToMerchant(event, "Invalid command. Add points with the command 'add <member ID> <points>'");

  const member = getMemberById(memberId);

  if (!member) {
    return replyToMerchant(event, `Member with ID ${memberId} not found`);
  }

  const pointToAdd = parseInt(pointToAddText);

  if (isNaN(pointToAdd)) {
    return replyToMerchant(event, "Please enter the number of points to add as a number only");
  }

  updatePointForMember(memberId, member.points + pointToAdd);
  pushToMember(member.uid, `You have received ${pointToAdd} points`);

  return replyToMerchant(event, `Added ${pointToAdd} points to member with ID ${memberId}`);
}

/**
 * Handle unknown events
 * @param {Object} event - Event to handle
 */
function handleUnknownEvent(event) {
  return replyToMerchant(event, "Sorry, I don't understand your command");
}

module.exports = router;

```

</details>
