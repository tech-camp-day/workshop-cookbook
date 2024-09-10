# Workshop 3: Create Your Own Bot/Build on Sample Projects

We have just seen the use of APIs and long-term data storage.
From now on, we can create more complex bots that store data for later use, call APIs from other sources to use the data.

There are 3 sample projects as follows:

- **Project 1: [Expense Memo](https://codesandbox.io/p/devbox/expense-memo-template-en-tqwrzm) - Expense Recording Bot**
  - Try it  
    [![Expense QR](/en/expense-en-qr.png)](https://lin.ee/yplyCvT)
  - Features in this project
    - Record expenses
    - Report total expenses within a specified period

  - Example features that can be added
    - Show the last 10 entries
    - Delete unwanted entries

- **Project 2: [Vocab Flash Card](https://codesandbox.io/p/devbox/vocab-flashcard-template-en-g3wlsl) - Vocabulary Guessing Game Bot**
  - Try it  
    [![Vocab QR](/en/vocab-en-qr.png)](https://lin.ee/B5h9G6fO)
  - Features in this project
    - Send English words for users to guess the Thai translation
  - Example features that can be added
    - Aggregate and report weekly scores
    - Allow repeated guesses if wrong, but give half points

- **Project 3: [Weather Bot](https://codesandbox.io/p/devbox/weather-bot-template-en-rj5jyl) - Weather Reporting Bot**
  - Try it  
    [![Weather QR](/en/weather-en-qr.png)](https://lin.ee/tYYjzsC)
  - Features in this project
    - Allow users to set their province
    - Report weather every morning
    - Alert if bad weather is expected in the next hour

  - Example features that can be added
    - Request immediate weather report
    - Notify when the rain will stop

- **[Blank Project](https://codesandbox.io/p/devbox/empty-line-chatbot-template-klx43w) - If you're skilled, go for it**
  - Includes LINE message receiver and sender
  - Includes a blank database creator

## Project Structure

All 3 projects have a similar structure. Since these are more complex projects, the code is divided into multiple files and folders as follows:

- `src/` - Main project folder
  - `index.js` - Main application file
  - `data/` - Folder related to data storage
    - `db.js` - Database connection + data management functions
    - Some projects will also have `.json` files as initial data for starting the system
  - `line/` - Folder related to sending messages via LINE
    - `messageHandler.js` - Receives messages from LINE, processes them, and sends back
    - `messageSender.js` - Helper for sending reply messages to LINE more easily
    - Some projects will also have `schedules.js` file to manage scheduled message sending
  - `routes/` - Folder related to server chatbot request handling
    - `line-webhook.js` - Handles requests related to LINE
  - Some projects have other folders such as `weather/` for managing weather data or others

## Setting Up the Project in Codesandbox

Since the projects in Workshop 3 are larger, setting up the project in Codesandbox has a few additional steps compared to Workshops 1 and 2 as follows:

0. Create a new bot following the steps in [this file](0_Create_LINE_bot.md)
1. Go to the link of the desired project
2. Click the "Fork" button at the top right corner of the page
3. After forking, the project you see on the screen will change to your own project
4. Click the "Share" button at the top right corner of the page
5. Click the "Move out of Drafts" button
6. In "Change permissions", select "Unlisted" to make the project accessible
7. Then set up Environment variables in Codesandbox by
  - Click the Codesandbox menu (square box at the top left corner) > Settings > Env Variables
  - Enter the following Environment variables:
    - `LINE_CHANNEL_ACCESS_TOKEN` - Channel access token from LINE Developers Console
    - `LINE_CHANNEL_SECRET` - Channel secret from LINE Developers Console
  - Click Restart microVM
8. In Codesandbox, the right half of the screen will be a Browser with a link at the top. Add `/webhook/line` to the end and paste the entire link into the "Webhook URL" field on the bot's "Messaging API" page _(Example: If the URL in Codesandbox is `https://blablabla.codesandbox.io`; change it to `https://blablabla.codesandbox.io/webhook/line` and put it in the Webhook URL)_
9. Click "Use webhook" if it is not already enabled
10. Make sure you configure your bot(or channel) following [this file](0_2_Config_LINE_bot.md)
