---
description: >-
  On this page we'll explain events, and how to handle them. We'll also see how
  to test events by "faking" their triggers!
---

# Events and Handlers

We already explored one event handler in [Your Basic Bot](https://github.com/AnIdiotsGuide/discordjs-bot-guide/tree/6d360a9eb88ca7eab83f6534bc0e042809aec1d2/getting-started/your-basic-bot.html), the `message` handler. Now let's take a look at some of the most important handlers that you will use, along with an example.

{% hint style="danger" %}
_**DO NOT NEST EVENTS**_ One important point: Do not nest any events \(aka "put one inside another"\). Ever. Events should be at the "root" level of your code, _beside_ the `message` handler and not within it.
{% endhint %}

## The `ready` event and its importance

Ah, asynchronous coding. So awesome. So hard to grasp when you first encounter it. The reality of discord.js and many, many other libraries you will encounter, is that code is not executed one line at a time, one after the other.

It should have been made obvious with the user of `client.on("message")` which triggers for each message. To explain how the `ready` event is important, let's look at the following code:

```javascript
const Discord = require("discord.js");
const client = new Discord.Client();

client.user.setActivity("Online!");

client.login("SuperSecretBotTokenHere");
```

This code will not work, because `client` is not immediately available after it's been initialized. `client.user` will be undefined in this case, even if we flipped the console.log and login lines. This is because it takes a small amount of time for discord.js to load its servers, users, channels, and all that jazz. The more servers the bot is on, the longer it takes.

To ensure that `client` and all its "stuff" is ready, we can use the `ready` event. Any code that you want to run on bootup that requires access to the `client` object, will need to be in this event.

Here's a simple example of using the `ready` event handler:

```javascript
client.on("ready", () => {
  client.user.setActivity(`on ${client.guilds.size} servers`);
  console.log(`Ready to serve on ${client.guilds.size} servers, for ${client.users.size} users.`);
});
```

## Detecting New Members

Another useful event is [`guildMemberAdd`](http://hydrabolt.github.io/discord.js/#!/docs/tag/indev/class/Client?scrollto=guildMemberAdd) which triggers whenever someone joins any of the servers the bot is on. You'll see this on smaller servers: a bot welcomes every new member in the \#welcome channel. The following code does this.

```javascript
client.on("guildMemberAdd", (member) => {
  console.log(`New User "${member.user.username}" has joined "${member.guild.name}"` );
  member.guild.channels.get("welcome").send(`"${member.user.username}" has joined this server`);
});
```

The objects available for each event are important: they're only available within these contexts. Calling `message` from the `guildMemberAdd` would not work - it's not in context. `client` is always available within all its callbacks, of course.

## Errors, Warn and Debug messages {#errors}

Yes, bots fail sometimes. And yes, the library can too! There's a little trick we can use, however, to prevent complete crashes sometimes: Capturing the `error` event.

The following small bit of code \(which can be anywhere in your file\) will catch all output message from discord.js. This includes all errors, warning and debug messages.

> **NOTE:** The debug event **WILL** output your token, so exercise caution when handing over a debug log.

```javascript
  client.on("error", (e) => console.error(e));
  client.on("warn", (e) => console.warn(e));
  client.on("debug", (e) => console.info(e));
```

## Testing Events {#testing}

So now you're wondering, how do I test those events? Do I have to join a server with an alternate account to test the guildMemberAdd event? Isn't that, like, super annoying?

Actually, there's an easy way to test almost any event. Without going into too many details, `client` , your Discord Client, extends something called the `EventHandler`. Any time you see `client.on("something")` it means you're handling an `event` called `"something"`. But EventHandler has another function other than `on`. It has `emit`. Emit is the counterpart for `on`. When you `emit` an event, it's handled by the callback for that event in `on`.

So what does it _mean_??? It means that if _you_ emit an event, your code can capture it. I know I know I'm rambling without giving you an example and you're here for examples. Here's one:

```javascript
client.emit("guildMemberAdd", message.member);
```

This emits the event that normally triggers when a new member joins a server. So it's _pretending_ like this particular member has rejoined the server even if they have not. This obviously works for any event but you have to provide the proper arguments for it. Since `guildMemberAdd` requires only a member, any member will do \(see [FAQ](../frequently-asked-questions.md) to know how to get another member\). I can trigger the `ready` event again by using `client.emit("ready")` \(the ready event does not take any parameter\).

What about other events? Let's see. `guildBanAdd` takes 2 parameters: `guild` and `user` , to simulate that a user was banned. So, you could `client.emit("guildBanAdd", message.guild, message.author)` to simulate banning the person sending a message. Again, getting those things \(Guilds and Users\) is in the FAQ.

You can do all this in a "test" command, or you can do what I do: use `eval`. [Check the Eval command](../examples/making-an-eval-command.md) when you're ready to go that route.

