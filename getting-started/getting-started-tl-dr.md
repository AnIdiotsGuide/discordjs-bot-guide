# Getting Started - TL;DR

{% hint style="info" %}
**This is the TL;DR version**. If you wish for a long version with more explanations, please see [this guide](getting-started-long-version.md)
{% endhint %}

## Create App and Bot Account

* Go to the [Discord.com Application Page](https://discord.com/developers/applications/me)
* Create a **New Application**, and give it a name
* Click **Bot**, **Add Bot** then finally click **Yes, do it**
* Visit `https://discord.com/oauth2/authorize?client_id=APP_ID&scope=bot` , replacing **APP\_ID** with the **Application ID** from the app page, to add the bot to your server \(or ask a server admin to do it for you\). If you're wanting slash commands as well, add `%20applications.commands` to the end of the URL above.
* Copy your bot's **Secret Token** and keep it for later

## Pre-requisite software

Depending on the operating system you're running the installation will be slightly different.

* nodejs \(Version 16.6 and higher required, see [Windows](https://nodejs.org/en/download/) or [Linux](https://nodejs.org/en/download/package-manager/)\)

Once you have this all installed, create a folder for your project and install discord.js:

`mkdir mybot` `cd mybot` `npm i discord.js`

## Example Code

The following is a simple ping/pong bot. Save as a text file \(e.g. `index.js`\), replacing the string on the last line with the secret bot token you got earlier:

```javascript
const { Client, Intents } = require("discord.js");
// Since discord.js exports an object by default, we can destructure it. Read up more here https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
 
client.on("ready", () => {
  console.log("I am ready!");
});
 
client.on("messageCreate", (message) => {
  if (message.content.startsWith("ping")) {
    message.channel.send("pong!");
  }
});
 
client.login("SuperSecretBotTokenHere");
```

## Launching the bot

In your command prompt, from inside the folder where `index.js` is located, launch it with:

`node index.js`

If no errors are shown, the bot should join the server\(s\) you added it to.

## Resources

* [Discord.js Documentation](http://discord.js.org) : For the love of all that is \(un\)holy, **read the documentation**. Yes, it will be alien at first if you are not used to "developer documentation" but it contains a whole lot of information about each and every feature of the API. Combine this with the examples above to see the API in context.
* [An Idiot's Guide](https://www.youtube.com/c/AnIdiotsGuide) is another great channel with more material. York's guides are great, and he continues to update them.
* [Evie.Codes on YouTube](https://www.youtube.com/channel/UCvQubaJPD0D-PSokbd5DAiw): If you prefer video to words, Evie's YouTube series \(which is good, though no longer maintained with new videos!\) gets you started with bots.
* [An Idiot's Guide Official Server](https://discord.gg/vXVxsAjSMF): The official server for An Idiot's Guide. Full of friendly helpful users!
* [Discord.js Official Server](https://discord.gg/bRCvFy9): The official server has a number of competent people to help you, and the development team is there too!
