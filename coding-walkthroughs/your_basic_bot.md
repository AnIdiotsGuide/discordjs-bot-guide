# Your First Bot

This chapter assumes you've followed the Getting Started chapter and your bot code compiles. Also, I have to repeat: if you don't understand the code you're about to see, coding a bot might not be for you. Go to [CodeAcademy](https://www.codecademy.com/learn/javascript) and learn Javascript.

In this chapter I'll guide you through the development of a simple bot with some useful commands. We'll start with the example we created in the first chapter:

```js
var Discord = require("discord.js");
var bot = new Discord.Client();

bot.on("message", msg => {
	if (msg.content.startsWith("ping")) {
		msg.channel.sendMessage("pong!");
	}
});

bot.on('ready', () => {
  console.log('I am ready!');
});

bot.login("yourcomplicatedBotTokenhere");
```

## Introducing Events

Before we dive into any further coding, we need to first understand what an *Event* is.

This is an event:
```js
bot.on("message", message => {
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
bot.on("message", msg => {
    if (msg.content.startsWith("ping")) {
      msg.channel.sendMessage("pong!");
    }

    else if (msg.content.startsWith("foo")) {
      msg.channel.sendMessage("bar!");
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
bot.on("message", msg => {

  // Set the prefix
  let prefix = "!";
  // Exit and stop if it's not there
  if(!msg.content.startsWith(prefix)) return;

  if (msg.content.startsWith(prefix + "ping")) {
    msg.channel.sendMessage("pong!");
  }

  else if (msg.content.startsWith(prefix + "foo")) {
    msg.channel.sendMessage("bar!");
  }
});
```

The changes to the code are still simple. Let's go through them:

- `let prefix = "!";` defines the prefix as the exclamation mark. You can change it to something else, of course.
- The line `if(!msg.content.startsWith(prefix)) return;` is a small bit of optimization which reads: "If the message does not start with my prefix, stop what you're doing. This prevents the rest of the function from running, making your bot faster and more responsive.
- The commands have changed so use this prefix, where `startsWith(prefix + "ping")` would only be triggered when the message starts with `!ping`.

The second point is just as important as having a single `message` event handler. Let's say the bot receives a hundred messages every minute (not much of an exaggeration on popular bots). If the function does not break off at the beginning, you are processing these hundred messages in each of your command conditions. If, on the other hand, you break off when the prefix is not present, you are saving all these processor cycles for better things. If commands are 1% of your messages, you are saving 99% processing power...

> OK I'm sorry, I'm bullshitting a little. It's not 99%, that is an exaggeration. It *is*, however, true that you save a ton on processor and RAM power.

## Preventing Botception
We're pretty much done with the basic bot. There's one last thing that I want to talk about: bots answering each other. Let's pretend for a moment that you have two bots on your server and each can respond to the same prefixed command, `!help`. But when that command is called, it replies: `!help commands: Type !help followed by one of the following to see details: ping , foo`.

Now, one person types `!help` in a channel, and both bots respond. But, they will also see the **other** bot saying `!help commands: [...]`, will see that as a request for help, answer each other... in an infinite loop. To prevent that from happening, we can add a second condition inside our `message` event handler, right below the one that checks for the prefix:

```js
bot.on("message", msg => {
  let prefix = "!";
  if(!msg.content.startsWith(prefix)) return;
  // our new check:
  if(msg.author.bot) return;  
  // [rest of the code]
});
```

That condition contains an *OR* operator, which reads as the following:

> If the ID of the author of this message is a bot", stop processing. This includes this bot, itself.

And now, we have a bot that only responds to 2 commands and does not waste any power trying to figure out anything else. Is this a complete basic bot? Sure! So let's end this page here and we'll take a look at some new concept next.

The full bot code would now be:

```js
var Discord = require("discord.js");
var bot = new Discord.Client();

bot.on("message", msg => {
  // Set the prefix
  let prefix = "!";
  // Exit and stop if it's not there
  if(!msg.content.startsWith(prefix)) return;
  // Exit if any bot
  if(msg.author.bot) return;  

  if (msg.content.startsWith(prefix + "ping")) {
    msg.channel.sendMessage("pong!");
  }

  else if (msg.content.startsWith(prefix + "foo")) {
    msg.channel.sendMessage("bar!");
  }
});

bot.login("yourcomplicatedBotTokenhere");
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
