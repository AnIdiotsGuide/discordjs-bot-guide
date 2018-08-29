---
description: >-
  So, you want to write a bot and you know some JavaScript, or maybe even
  node.js. You want to do cool things like a tag commands, image searches, the
  whole shebang. Well you're at the right place!
---

# Getting Started - Linux TL;DR

{% hint style="info" %}
**This is the TL;DR version for Linux**. If you wish for a long version with more explanations, please see [this guide](getting-started-long-version.md)
{% endhint %}

## Create App and Bot Account

* Go to the [Discordapp.com Application Page](https://discordapp.com/developers/applications/me)
* Create a **New Application**, and give it a name
* Click **Create a bot account**, then **Yes, do it**
* Visit `https://discordapp.com/oauth2/authorize?client_id=APP_ID&scope=bot` , replacing **APP\_ID** with the **Client/Application ID** from the app page, to add the bot to your server \(or ask a server admin to do it for you\).
* Copy your bot's **Token** and keep it for later

## Pre-requisite software

Install the following through your package manager:

* nodejs \(Version 6.X and higher required, see [here](https://nodejs.org/en/download/package-manager/)\)

Once you have this all installed, create a folder for your project and install discord.js:

`mkdir mybot && cd mybot` `npm install discord.js`

**For sound support** add `npm install opusscript` \(ez mode\) or `npm install node-opus` \(better performance but requires `python 2.7.x` and `build-essential`\). BOTH these options require `ffmpeg` to run on your system, installed through `sudo apt-get install ffmpeg`.

## Example Code

The following is a simple ping/pong bot. Save as a text file \(e.g. `mybot.js`\), replacing the string on the last line with the secret bot token you got earlier:

```javascript
const Discord = require("discord.js");
const client = new Discord.Client();

client.on("ready", () => {
  console.log("I am ready!");
});

client.on("message", (message) => {
  if (message.content.startsWith("ping")) {
    message.channel.send("pong!");
  }
});

client.login("SuperSecretBotTokenHere");
```

## Launching the bot

In your terminal, from inside the folder where `mybot.js` is located, launch it with:

`node mybot.js`

If no errors are shown, the bot should join the server\(s\) you added it to.

## Resources

* [Discord.js Documentation](http://discord.js.org) : For the love of all that is \(un\)holy, **read the documentation**. Yes, it will be alien at first if you are not used to "developer documentation" but it contains a whole lot of information about each and every feature of the API. Combine this with the examples above to see the API in context.
* [An Idiot's Guide](https://www.youtube.com/c/AnIdiotsGuide) is another great channel with more material. York's guides are great, and he continues to update them.
* [Evie.Codes on Youtube](https://www.youtube.com/channel/UCvQubaJPD0D-PSokbd5DAiw): If you prefer video to words, Evie's youtube series \(which is good, though no longer maintained with new videos!\) gets you started with bots.
* [An Idiot's Guide Official Server](https://discord.gg/9ESEZAx): The official server for An Idiot's Guide. Full of friendly helpful users!
* [Discord.js Official Server](https://discord.gg/bRCvFy9): The official server has a number of competent people to help you, and the development team is there too!

