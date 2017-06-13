# Understanding Events and Handlers

We already explored one event handler in [Your Basic Bot](your-basic-client.html), the `message` handler. Now let's take a look at some of the most important handlers that you will use, along with an example.

> **Don't nest events**  
> One important point: Do not nest any events within others unless you know what you're doing. Events should be at the "root" level of your code, _beside_ the `message` handler and not within it.

## The `ready` event and its importance

Ah, asynchronous coding. So awesome. So hard to grasp when you first encounter it. The reality of discord.js and many, many other libraries you will encounter, is that code is not executed one line at a time, one after the other.

It should have been made obvious with the user of `client.on("message")` which triggers for each message. To explain how the `ready` event is important, let's look at the following code:

```js
const Discord = require("discord.js");
const client = new Discord.Client();

console.log(client.user.id);

client.login("SuperSecretBotTokenHere");
```

This code will not work, because `client` is not immediately available after it's been initialized. `client.user` will be undefined in this case, even if we flipped the console.log and login lines. Well it might work - sometimes. This is because it takes a small amount of time for discord.js to load its servers, users, channels, and all that jazz. The more servers the bot is on, the longer it takes.

To ensure that `client` and all its "stuff" is ready, we can use the `ready` event. Any code that you want to run on bootup that requires access to the `client` object, will need to be in this event.

Here's a simple example of using the `ready` event handler:

> The sizes for Collections \(like channels and users\) depends on the `fetchAllMembers: true` option in the client.

```js
client.on("ready", () => {
  console.log(`Ready to serve in ${client.channels.size} channels on ${client.guilds.size} servers, for a total of ${client.users.size} users.`);
});
```

I have this in all my bots, in various forms. If you need to loop across all your servers, this is also where you would do it.

## Detecting New Members

Another useful event is [`guildMemberAdd`](http://hydrabolt.github.io/discord.js/#!/docs/tag/indev/class/Client?scrollto=guildMemberAdd) which triggers whenever someone joins any of the servers the bot is on. You'll see this on smaller servers: a bot welcomes every new member in the \#general channel. The following code does this.

```js
client.on("guildMemberAdd", (member) => {
  console.log(`New User "${member.user.username}" has joined "${member.guild.name}"` );
  member.guild.defaultChannel.send(`"${member.user.username}" has joined this server`);
});
```

The objects available for each event are important: they're only available within these contexts. Calling `message` from the `guildMemberAdd` would not work - it's not in context. `client` is always available within all its callbacks, of course.

## Errors, Warn and Debug messages

Yes, bots fail sometimes. And yes, the library can too! There's a little trick we can use, however, to prevent complete crashes sometimes: Capturing the `error` event.

The following small bit of code \(which can be anywhere in your file\) will catch all output message from discord.js. This includes all errors, warning and debug messages.

> **NOTE:** The debug event **WILL** output your token, so exercise caution when handing over a debug log.

```js
  client.on("error", (e) => console.error(e));
  client.on("warn", (e) => console.warn(e));
  client.on("debug", (e) => console.info(e));
```



