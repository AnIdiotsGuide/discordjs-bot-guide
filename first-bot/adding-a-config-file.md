---
description: >-
  On this page we'll be adding a configuration file where you can store your
  token and a few other useful things, so you can more easily share your code or
  duplicate it.
---

# Adding a Config File

Now that you have a bot up and running, we can start splitting it into some more useful parts. And the first part of this is separating some of the variables we have defined into a configuration file, `config.json`. We'll be loading this file on boot.

{% hint style="danger" %}
Putting your token in a config file is fine, but **DO NOT COMMIT IT TO GITHUB** or any other public location. Usually, adding a `.gitignore` file to your project should be enough. [Here's an example](https://github.com/github/gitignore/blob/master/Node.gitignore). Simply add a line that says `config.json` to that file, save it in your project root and you should be good. [More Details in this video](https://www.youtube.com/watch?v=iyNIHQkVGao), and [this page](../other-guides/using-git-to-share-and-update-code.md).
{% endhint %}

## Why a config file?

One of the advantages of having a configuration file is that you can safely copy your bot's code into, say, hastebin.com to show people, and your token won't be in there. A second advantage is that you can upload the code to a repository like github and, as long as you ignore the config file, your bot can be shared but remain secure. We'll see that in action in a future walkthrough.

## Step 1: The config file

The 2 things that we can add to the config file are:

* The bot's token
* The prefix

Simply take the following example, and create a new file in the same folder as your bot's file, calling it `config.json`:

```javascript
{
  "token": "insert-bot-token-here",
  "prefix": "!"
}
```

## Step 2: Require the config file

At the top of your bot file, you need to add a line that will load this configuration, and put it in a variable. This is what it looks like:

```javascript
const Discord = require("discord.js");
const client = new Discord.Client();
const config = require("./config.json");
```

This means that now, `config` is your configuration object. `config.token` is your token, `config.prefix` is your prefix! Simple enough.

## Step 3: Using `config` in your code

So let's use what we did earlier, and use the token from the config file, instead of putting it directly in the file. The last line of our bot looks like this:

```javascript
client.login("SuperSecretBotTokenHere");
```

And we simply need to change it to this:

```javascript
client.login(config.token);
```

The other thing we have, is of course the prefix. Again from before, we have this line in our message handler:

```javascript
const prefix = "!";
client.on("message", (message) => {
  if (!message.content.startsWith(prefix) || message.author.bot) return;

  if (message.content.startsWith(prefix + "ping")) {
    message.channel.send("pong!");
  } else
  if (message.content.startsWith(prefix + "foo")) {
    message.channel.send("bar!");
  }
});
```

We're using `prefix` in a few places, so we need to change them all. Here's how it looks like after the changes:

```javascript
client.on("message", (message) => {
  if (!message.content.startsWith(config.prefix) || message.author.bot) return;

  if (message.content.startsWith(config.prefix + "ping")) {
    message.channel.send("pong!");
  } else
  if (message.content.startsWith(config.prefix + "foo")) {
    message.channel.send("bar!");
  }
});
```

{% hint style="info" %}
The removal of the line that sets the prefix. We don't need it anymore!
{% endhint %}

## Changing the config

If you're asking yourself "but how do I change the prefix, now?" fear not, we have some help for you. We suggest you start by reading the rest of this section of the guide \("First Bot"\) and then hop on to the [Per-Server Configuration Guide on the Enamp Documentation](https://enmap.evie.codes/examples/settings)!

## Extending the idea

So is there anything else you could put in that config file? Absolutely. One thing I use it for is to store my personal user ID, so that my bot can use it to recognize me and give me exclusive access to some commands.

```javascript
{
  "token": "insert-bot-token-here",
  "prefix": "!",
  "ownerID": "your-user-ID"
}
```

Then, in a protected command, I could use the following line to prevent access to all the users that think they can use it!:

```javascript
if(message.author.id !== config.ownerID) return;
```

## What's next _now_?

So now that we have a functional bot with a configuration file, let's add more stuff to it! Follow me to the [Command with arguments](command-with-arguments.md) for the next part!

