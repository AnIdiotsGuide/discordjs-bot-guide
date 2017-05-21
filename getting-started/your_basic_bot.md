#Your First Bot

This chapter assumes you've followed the Getting Started chapter and your bot code compiles. Also, I have to repeat: if you don't understand the code you're about to see, coding a bot might not be for you. Go to [CodeAcademy](https://www.codecademy.com/learn/javascript) and learn Javascript.

In this chapter I'll guide you through the development of a simple bot with some useful commands. We'll start with the example we created in the first chapter:

```js
const Discord = require('discord.js');
const client = new Discord.Client();

client.login('SuperSecretBotTokenHere');

client.on('ready', () => {
  console.log('I am ready!');
});

client.on('message', (message) => {
  if (message.content.startsWith('ping')) {
    message.channel.send('pong!');
  }
});
```

## Introducing Events

Before we dive into any further coding, we need to first understand what an *Event* is.

This is an event:
```js
client.on('message', (message) => {
  // This code runs when the event is triggered
});
```

This is, specifically, an event in *discord.js* but it's similar to how other APIs handle events. This event triggers *every time the bot sees a message*. This includes every channel the bot has access to as well as any direct or private message it receives. If someone sends 5 messages on a channel, this event fires 5 times.

Why is this important? Well, if you intend to use your bot on a large server, or if you want it to be on multiple servers, this becomes a large number of events triggering at every moment. I don't want to go into too much optimization talk, but for a single point: **use a single event function for each event**.

Discord.js contains a large number of events that can trigger under certain situations. For instance, the `ready` event triggers when the bot comes online. The `guildMemberAdd` event triggers when a new user joins a server shared with the bot. For a full list of events, see [Events in the documentation](http://discordjs.readthedocs.io/en/latest/docs_client.html#events). We will come back to some of those later in this chapter.

## Adding a second command

One of the first useful things you might want to learn is how to add a second command to your bot. While there are *better* ways than what I'm about to show you, for the time being this will be enough.

> From now on I will omit the code that requires and initiates the discord.js and concentrate on specific parts of the code.

```js
client.on('message', (message) => {
  if (message.content.startsWith('ping')) {
    message.channel.send('pong!');
  } else

  if (message.content.startsWith('foo')) {
    message.channel.send('bar!');
  }
});
```

Save your code and reload your bot. To do so, use `CTRL+C` in the command line, and re-run `node mybot.js`. Yes, there are better ways to reload the code, as you will see later in this book.

You can test your new command by saying `foo` in a channel you share with the bot. You can also confirm that `ping` still returns `pong`!

## Using a Prefix

You might have noticed that a lot of bots respond to commands that have a prefix. This might be an exclamation mark (!), a dot (.), a question mark(?), or another character. This is useful for two things.

First, if you don't use a unique prefix and have more than one bot on a server, both will respond to the same commands. On developer servers, typing `!help` leads to a flood of replies and private messages which is something to avoid.

Second, in the example above we respond when the message *starts with* the 3 characters, `foo`. In its current state, this means the following sentence will trigger the bot's response: **fool, you have not heard the last of me!**. Yes, that's an odd example, but it's still valid - say this on your bot's channel and he will respond.

To work around this, we'll be using prefix, which we will store in a variable. This way we get the prefix as well as the ability to change it for all commands in one place. Here's an example code that does this:

```js
client.on('message', (message) => {
  // Set the prefix
  let prefix = '!';

  // Exit and stop if it's not there
  if (!message.content.startsWith(prefix)) return;

  if (message.content.startsWith(prefix + 'ping')) {
    message.channel.send('pong!');
  } else
  if (message.content.startsWith(prefix + 'foo')) {
    message.channel.send('bar!');
  }
});
```

The changes to the code are still simple. Let's go through them:

- `let prefix = '!';` defines the prefix as the exclamation mark. You can change it to something else, of course.
- The line `if(!msg.content.startsWith(prefix)) return;` is a small bit of optimization which reads: 'If the message does not start with my prefix, stop what you're doing. This prevents the rest of the function from running, making your bot faster and more responsive.
- The commands have changed so use this prefix, where `startsWith(prefix + 'ping')` would only be triggered when the message starts with `!ping`.

The second point is just as important as having a single `message` event handler. Let's say the bot receives a hundred messages every minute (not much of an exaggeration on popular bots). If the function does not break off at the beginning, you are processing these hundred messages in each of your command conditions. If, on the other hand, you break off when the prefix is not present, you are saving all these processor cycles for better things. If commands are 1% of your messages, you are saving 99% processing power...

> OK I'm sorry, I'm bullshitting a little. It's not 99%, that is an exaggeration. It *is*, however, true that you save a ton on processor and RAM power.

## Preventing Botception
We're pretty much done with the basic bot. There's one last thing that I want to talk about: bots answering each other. Let's pretend for a moment that you have two bots on your server and each can respond to the same prefixed command, `!help`. But when that command is called, it replies: `!help commands: Type !help followed by one of the following to see details: ping , foo`.

Now, one person types `!help` in a channel, and both bots respond. But, they will also see the **other** bot saying `!help commands: [...]`, will see that as a request for help, answer each other... in an infinite loop. To prevent that from happening, we can add a second condition inside our `message` event handler, right below the one that checks for the prefix:

```js
client.on('message', (message) => {
  let prefix = '!';
  // our new check:
  if (!message.content.startsWith(prefix) || message.author.bot) return;
  // [rest of the code]
});
```

That condition contains an *OR* ( || ) operator, which reads as the following:

> If there is no prefix or the author of this message is a bot, stop processing. This includes this bot, itself.

And now, we have a bot that only responds to 2 commands and does not waste any power trying to figure out anything else. Is this a complete basic bot? Sure! So let's end this page here and we'll take a look at some new concept next.

The full bot code would now be:

```js
const Discord = require('discord.js');
const client = new Discord.Client();

client.login('SuperSecretBotTokenHere');

client.on('message', (message) => {
  // Set the prefix
  let prefix = '!';
  // Exit and stop if the prefix is not there or if user is a bot
  if (!message.content.startsWith(prefix) || message.author.bot) return;

  if (message.content.startsWith(prefix + 'ping')) {
    message.channel.send('pong!');
  } else
  if (message.content.startsWith(prefix + 'foo')) {
    message.channel.send('bar!');
  }
});
```

#Adding a `config.json` file to your bot?

Now that you have a bot up and running, we can start splitting it into some more useful parts. And the first part of this is separating some of the variables we have defined into a configuration file, `config.json`. We'll be loading this file on boot.

## Why a config file?

One of the advantages of having a configuration file is that you can safely copy your bot's code into, say, hastebin.com to show people, and your token won't be in there. A second advantage is that you can upload the code to a repository like github and, as long as you ignore the config file, your bot can be shared but remain secure. We'll see that in action in a future walkthrough.

## Step 1: The config file

The 2 things that we can add to the config file are:

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
const Discord = require('discord.js');
const client = new Discord.Client();
const config = require('./config.json');
```

This means that now, `config` is your configuration object. `config.token` is your token, `config.prefix` is your prefix! Simple enough.

## Step 3: Using `config` in your code

So let's use what we did earlier, and use the token from the config file, instead of putting it directly in the file. The last line of our bot looks like this:

```js
client.login('SuperSecretBotTokenHere');
```

And we simply need to change it to this:

```js
client.login(config.token);
```

The other thing we have, is of course the prefix. Again from before, we have this line in our message handler:

```js
client.on('message', (message) => {
  let prefix = '!';

  if (!message.content.startsWith(prefix) || message.author.bot) return;

  if (message.content.startsWith(prefix + 'ping')) {
    message.channel.send('pong!');
  } else
  if (message.content.startsWith(prefix + 'foo')) {
    message.channel.send('bar!');
  }
});
```

We're using `prefix` in a few places, so we need to change them all. Here's how it looks like after the changes:

```js
client.on('message', (message) => {
  if (!message.content.startsWith(config.prefix) || message.author.bot) return;

  if (message.content.startsWith(config.prefix + 'ping')) {
    message.channel.send('pong!');
  } else
  if (message.content.startsWith(config.prefix + 'foo')) {
    message.channel.send('bar!');
  }
});
```

> Note the removal of the line that sets the prefix. We don't need it anymore!

## Changing the config

You're probably wondering 'But how do I modify my prefix with a command?', right? You're in luck, that part is actually fairly easy!

This requires, first of all, the native `fs` module. At the top of your bot file, add:
>NOTE `fs` is a native module, you do **not** need to install it.

```js
const fs = require('fs')
```

Now, let's say you wanted a prefix-changing command. This would take the shape of:

```js
if(message.content.startsWith(config.prefix + 'prefix')) {
  // Gets the prefix from the command (eg. "!prefix +" it will take the "+" from it)
  let newPrefix = message.content.split(' ').slice(1, 2)[0];
  // change the configuration in memory
  config.prefix = newPrefix;

  // Now we have to save the file.
  fs.writeFile('./config.json', JSON.stringify(config), (err) => if(err) console.error(err));
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
  "ownerID": "your-user-ID"
}
```

Then, in the [eval command](/samples/making-an-eval-command.md), I could use the following line to prevent access to all the users that think they can use my eval!:

```js
if(message.author.id !== config.ownerID) return;
```

## What's next *now*?

Once you're done with this basic bot, there's a couple of things you should take a look at in this guide.

### References and Information
- [Understanding Events and Handlers](events_and_handlers.md) gives you more details about events (things that trigger code) and handlers (the code that triggers).
- [Understanding Roles](understanding_roles.md) is an important overview of Roles and Collections.

### More code samples
For more code samples you can use in your bot, check out:

- [Welcome Message every X users](../samples/welcome_message_every_x_users.md) : it's easy to welcome users as they come in, but can you do it every 10 users? I'll show you how.
- [Message Reply Array](../samples/message_reply_array.md) : instead of a dozen conditions with simple text replies, we'll use an array to make things easier to look at and add to!
- [Command with Arguments](../samples/command_with_arguments.md) : How do I do `!echo a message` or `!kick @xXx_phantom_sniper_xXx`? Follow this to learn how.
- [Selfbots, the Awesomest thing in the universe](../samples/selfbots_are_awesome.md) : If you want to use lenny make personal tags just like me, follow along, young apprentice.
