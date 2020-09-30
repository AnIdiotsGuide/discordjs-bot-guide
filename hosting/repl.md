---
description: >-
  Let's take a look at how to host a bot on repl.it securely and correctly. And,
  for free!
---

# Hosting on Repl.it

There have been multiple attempts for people to find free ways to host their bot through the years that this guide has been around. Unfortunately it seems every time we find a new one, it lasts for a while and then somehow locks up. [This is what happened to the Glitch app](glitch.md), but let's hope Repl.it lasts for a while longer.

This page will guide you through setting up your repl.it app, creating a discord.js bot that stays online 24/7. This method, unlike what happened to glitch, [_has been approved by the repl.it developers_](https://github.com/AnIdiotsGuide/discordjs-bot-guide/issues/119)_._ In other words, you're not breaking any rule doing this to keep your bot online.

{% hint style="danger" %}
Every hosting service has their downside - Repl.it's limitation is that you cannot hide files from the public eye on free accounts, except for the .env file. This means, if you were to use a file-based or sqlite-based database \(such as enmap, quick.db, nedb\) your files would be visible. Since this is against the Discord Terms of Service \(exposing potential user data\), _do not use these modules or json files_ to store data. Instead, you can use the [Repl.It Database](https://docs.repl.it/misc/database) system to store data, or an external database server like Atlas Mongo, or Firebase.
{% endhint %}

## Creating your Repl.it Account

This part is pretty self-explanatory. You can simply head on over to the [Repl.it main site](https://repl.it/), then click on the **Signup** button at the top-right of the page. You can create your account manually with a username, password, and email, but you can also click the Github, Google or Facebook icon to authenticate with those services. 

Personally, we recommend using the Github login, as it will give you easier integration into Github repositories. This is an optional, but easier way to setup your bot.

## Creating your bot from a Github repository

The easiest way to get started using Repl.it is to create your bot from an existing codebase hosted on github. This takes only a few moments, but it does require that code to be available. 

To create a new repl from a Github repo, simply click the blue **+** icon at the top-right of the Repl.it page \(you have to be logged in, of course\) and then switch to the **Import from Github** page. Here, you can select from one of your own repositories if you have one ready. If you _don't_, don't fret, we have a few available for you! Simply paste one of the following URLs in the box to get the appropriate starter template. All the templates are pre-configured to work directly with .env files so they're safe to use as-is:

* [https://github.com/eslachance/bf-basic](https://github.com/eslachance/bf-basic) : A very basic bot with a single file and a ping/pong command. Basically, an adapted example from the official discord.js docs.
* [https://github.com/eslachance/bf-arguments](https://github.com/eslachance/bf-arguments) : A bot that's still a single file, but includes our suggested Argument system already. Quick to start with some basic code just to get you started easily.
* [https://github.com/eslachance/djs-handler](https://github.com/eslachance/djs-handler.git) : A bot with our recommended Command Handler, where each file and event is its own file. I'd recommend going with this one, the code is still very easy to understand. Heck, it even includes a Help command and support for commands in folders for easy categorization!
* [https://github.com/AnIdiotsGuide/guidebot/](https://github.com/AnIdiotsGuide/guidebot/) : The full guidebot boilerplate. Note that this one will not work as-is, so we'll have a separate page to give you the exact steps to make this one work \(I'll write it after this!\). Notably however, this repository uses enmap which creates a data folder which, as the warning above implies, is public. We'll update the page once we have more info on this.

Once you've entered your repository \(it will be simplified to `username/repositoryname`, don't worry, it's normal\), just click **Import from Github** and watch it go! Once that's done, you'll be presented with a small window on the top-right of the workspace. This window asks you _what language your repl is using_. If nodejs isn't already selected \(it tries to guess\), select it. Leave the run button to `npm start` and just click Done.

![](../.gitbook/assets/image%20%285%29.png)

Once it's finished you'll be ready to Run the bot, but _don't do this right now_, as we need to make sure your token is there and secured \(see below\).

## Create your bot from scratch

The other way to create a new bot is to start from an empty repl and create your own code. This is still fairly simple, and you can follow along the [First Bot](../first-bot/your-first-bot.md) section of this guide to create your code \(or just do your thing and make your own!\). 

To create a new empty repl, just click the **+** icon at the top-right of the Repl.it page, then select the **Nodejs** language \(Repl.it supports _a lot of languages_ , it's very cool and useful\). Give your repl a name or leave the autogenerated name, then hit **Create Repl**. This will take a few seconds then you're ready to proceed.

There's already an index.js file created here, so you can throw your code in here, and hit Run at the top to start the bot. _Don't do this right now_, you need to setup the token correctly inside the secured .env file!

## Adding your token to the .env file and using it

If you've used the bf-basic and bf-handler repositories, the setup is already done for you and all you need to do is to add your token in the .env file to get started. If not, please follow along.

{% hint style="info" %}
The .env file \(exactly as-is, not `tokens.env` or `thing.env`!\) is the [_Environment Variables File_](../other-guides/env-files.md). It's used to simulate environment variables at the operating system level, but only for your specific app. It's named `.env` because of it's linux-related origins, where files starting with a dot \("dot files"\) are considered "hidden" and secret. They're usually more secure and only visible to those with proper permissions. In repl.it, the .env file is completely hidden from view from the public unless you SHARE the repository with full rights, or start a MULTIPLAYER SESSION for others to code with this. This means, DO NOT share or start a multiplayer session with someone you don't trust with your bot token!
{% endhint %}

To setup your bot to use a token in the .env file, start by creating one. Click the _Add File_ button on top of the file list, call the file `.env` \(exactly, no "filename" here!\), and it will open in the editor.

In the file, you need to add the variables you want hidden, in the format of `VARIABLENAME=VALUE`. Note that there shouldn't be any spaces or quotes here, this isn't javascript! For example, an env file with a fake token in it, the owner's ID, and a bot prefix \(~\):

```text
CLIENT_TOKEN=MTg-this-IzNzU3OTA5NjA-is.not-DCeFB-a.real-r4DQlO-t0ken-qerT0
OWNER_ID=139412744439988224
PREFIX=~
```

With that entered \(no need to save, Repl.it does that on its own\), let's modify the index.js file to use that token correctly. To access environment variables, you simply use `process.env.VARIABLENAME` \(the case must match, variable names are always UPPERCASE\). 

Note that discord.js automatically reads the CLIENT\_TOKEN variable so you don't even need to refer to it at all in your code. 

So, to access the Owner ID, you'd use `process.env.OWNER_ID`. A command locked to the owner would look like this: 

```javascript
if(command === 'secured') {
  if(message.author.id !== process.env.OWNER_ID) {
    return message.reply("You're not the boss of me now!");
  }
  // your super secure owner-only command
}
```

Another good example is with the prefix. Our [Arguments ](../first-bot/command-with-arguments.md)code would look like this: 

```javascript
if (!message.content.startsWith(process.env.PREFIX)) return;
const args = message.content.slice(process.env.PREFIX.length).trim().split(/ +/g);
const command = args.shift().toLowerCase();
```

{% hint style="info" %}
Our [Environment Variables](../other-guides/env-files.md) guide suggests the use of the `dotenv` module to read these values. With repl.it you don't need this module - env vars are directly sent to your app and are readable directly!
{% endhint %}

## Running the Bot

So, once you have your code and everything is ready, the next step is to start the bot. Let's start 'er up!

To run, simply click the "Run &gt; " button at the top of the editor. Now you might be wondering, "But Evie, what about installing discord.js, how do I do that?" and it's a fair question - but moot. Repl.it automatically installs any dependency that you code calls, on demand. This means, when you require discord.js and start the bot, it will automatically install the latest version of the library. It's almost like magic!

If you get an "Invalid token" error, check your .env file and make sure of the following: 

* It's called `.env` exactly, don't be fancy here, the name must be _exact_.
* Make sure the format is exactly `CLIENT_TOKEN=YOURTOKENHERE`, you must not add quotes, spaces, or any other characters than the variable name, and equal sign, then the token directly.
* Make sure you're using your bot's token and not the client secret or its ID.

## Keeping the bot awake

There are two things that you need to do in order for the bot to stay awake. You need a web server running, and you also need to ping the site on a regular interval. Again, don't worry, [this method is fully approved by the Repl.it developers!](https://github.com/AnIdiotsGuide/discordjs-bot-guide/issues/119)

### The Web Server

Our glitch guide has this using express, but honestly you don't need it. Here's a very basic web server using node.js' own `http` module, it's really all you need. Place this in your index.js file, outside of any event handler \(in other words, at the very bottom of the file\): 

```javascript
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200);
  res.end('ok');
});
server.listen(3000);
```

That's it! That's a web server! Make sure to RUN your app before continuing. If all goes well, a new panel appears on the right: it has a new URL ending with `.co` and is all white except for the "ok" sent by the server. 

### The automatic ping

So, the first thing you'll need is to register a new account on UptimeRobot.com. I'll let you do that on your own, it's not rocket science! 

* Once you have the account, head on over to [your dashboard](https://uptimerobot.com/dashboard).
* Click on **Add New Monitor** at the top-left of the screen.
* Set the _Monitor Type_ as **HTTP\(s\)**. 
* Give it a friendly name \("My Repl Site" or whatever\)
* The URL that you enter is the one at the top of the _white window seen above the console_ in your repl. it should end with a `.repl.co` domain name.
* Set the timer to 5 minutes \(the lowest you can on a free plan\).

If all worked well, you'll see the ping turn green \(if it doesn't, make sure to RUN your application!\)

![](../.gitbook/assets/image%20%286%29.png)

## Conclusion

While hosting your bot for free on Repl.it is not the _best_ solution out there \(we recommend going to a small-priced VPS such as [OVH.com](https://www.ovh.com/)\), we understand that some new developers can't afford even a few dollars a month. 

For Repl.it support, please visit[ their discord server](https://discord.com/invite/5gcPC6B). An Idiot's Guide can provide very limited support for external services, and Repl.it is not an exception to this rule!

