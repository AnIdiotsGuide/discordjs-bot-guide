#Adding a `config.json` file to your bot?

Now that you have a bot up and running, we can start splitting it into some more useful parts. And the first part of this is separating some of the variables we have defined into a configuration file, `config.json`. We'll be loading this file on boot.

## Why a config file?

One of the advantages of having a configuration file is that you can safely copy your bot's code into, say, hastebin.com to show people, and your token won't be in there. A second advantage is that you can upload the code to a repository like github and, as long as you ignore the config file, your bot can be shared but remain secure. We'll see that in action in a future walkthrough.

## Step 1: The config file

So let's say we have exactly our code from [Your First Bot](/coding-walkthroughs/your_basic_bot.md). The 2 things that we can add to the config file are:

* The bot's token
* The prefix

Simply take the following example, and create a new file in the same folder as your bot's file, calling it `config.json`:

```json
{
  "token": "insert-bot-token-here",
  "prefix": "!"
}
```

## Step 2: Require the config file

At the top of your bot file, you need to add a line that will load this configuration, and put it in a variable. This is what it looks like:

```js
const Discord = require("discord.js");
const client = new Discord.Client();
const config = require('./config.json');
```

This means that now, `config` is your configuration object. `config.token` is your token, `config.prefix` is your prefix! Simple enough.

## Step 3: Using `config` in your code

So let's use what we have in [Your First Bot](/coding-walkthroughs/your_basic_bot.md), and use the token from the config file, instead of putting it directly in the file. The last line of our bot looks like this:

```js
client.login("SuperSecretBotTokenHere");
```

And we simply need to change it to this:

```js
client.login(config.token);
```

The other thing we have, is of course the prefix. Again from [Your First Bot](/coding-walkthroughs/your_basic_bot.md), we have this line in our message handler:

```js
const prefix = "!";
client.on("message", (message) => {
  if(!message.content.startsWith(prefix)) return;

  if (message.content.startsWith(prefix + "ping")) {
    message.channel.send("pong!");
  } else

  if (message.content.startsWith(prefix + "foo")) {
    message.channel.send("bar!");
  }
});
```

We're using `prefix` in a few places, so we need to change them all. Here's how it looks like after the changes:

```js
client.on("message", (message) => {
  if(!message.content.startsWith(config.prefix)) return;

  if (message.content.startsWith(config.prefix + "ping")) {
    message.channel.send("pong!");
  } else

  if (message.content.startsWith(config.prefix + "foo")) {
    message.channel.send("bar!");
  }
});
```

> Note the removal of the line that sets the prefix. We don't need it anymore!

## Changing the config

You're probably wondering "But how do I modify my prefix with a command?", right? You're in luck, that part is actually fairly easy!

This requires, first of all, the `fs` module. At the top of your bot file, add:

```js
const fs = require("fs")
```

Now, let's say you wanted a prefix-changing command. This would take the shape of:

```js
if(message.content.startsWith(config.prefix + "prefix"))
  // get arguments for the command, as: !prefix +
  let args = message.content.split(" ").slice(1);
  // change the configuration in memory
  config.prefix = args[0];

  // Now we have to save the file.
  fs.writeFile('./config.json', JSON.stringify(config), (err) => {if(err) console.error(err)});
}
```

> If you want to understand what `args` is, please read [Command with arguments](/samples/command_with_arguments.md).

Awesome! Now the configuration has been changed, and we've edited config.json so that next time the bot restarts, the new prefix will be used! There's a _lot_ more you can do with JSON files though, for more of this check out : [Storing Data in a JSON file](/storing-data-in-a-json-file.md). This example does an awesome _points_ system just like the horrible Mee6 bot.

## Extending the idea

So is there anything else you could put in that config file? Absolutely. One thing I use it for is to store my personal user ID, so that my bot can use it to recognize me and give me exclusive access to some commands. For example take a look at [the eval command](/samples/making-an-eval-command.md), which locks eval to my user ID.

```json
{
  "token": "insert-bot-token-here",
  "prefix": "!",
  "ownerID": "139412744439988224"
}
```

Then, in the [eval command](/samples/making-an-eval-command.md), I could use the following line to prevent access to all the plebs that think they can use my eval!:

```js
if(message.author.id !== config.ownerID) return;
```

Awesome! Now go back to coding!
