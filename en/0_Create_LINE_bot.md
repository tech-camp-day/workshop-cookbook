# How to Create a LINE Bot

1. Log in to LINE at the [LINE Developers Console](https://developers.line.biz/en/)
2. Click on "Create a new provider" and fill in the required information _(If you already have a provider, you can skip this step)_
3. Click on "Create a new channel" and select "Messaging API"
4. Click on "Create a LINE Official Account.
5. Fill in the required information and click "Create".
6. Once you reach 3rd Step, scroll down and click "Later".
7. Click "Settings" at the top right corner.
8. Click "Messaging API", and then click "Enable Messaging API"
9. Select the provider that we just created from 1.

At this point, you have created your LINE Bot, but you need to complete some additional settings:

1. On the Bot's Channel page, go to the "Messaging API" tab.
2. Scroll down to "Greeting messages", click Edit on the right side to open the Response settings page.
3. Configure the settings as follows:
   - Chat: Off
   - Greeting message: Off
   - Webhooks: **On**
   - Auto-response messages: Off
   - Response hours: Off
4. Close the Response settings page, return to the previous page, scroll down to the bottom, and click "Issue" under Channel access token.

Your Bot is now ready for use.
