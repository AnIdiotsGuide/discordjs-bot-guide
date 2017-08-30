# Frequently Asked Questions

In this page, some very basic, frequently-asked questions are answered. It's important to understand that **these examples are generic** and will most likely not work if you just copy/paste them in your code. You need to **understand** these lines, not just blindly shove them in your code.

## Code Examples

## Bot and Bot Client

```js
// Set the bot's "Playing: " status (must be in an event!)
client.on("ready", () => {
    client.user.setGame("with my code");
});

// NOTE: CURRENTLY BROKEN IN DISCORD.JS 9 THROUGH 11.1. USE THIS INSTEAD:
client.on("ready", () => {
    client.user.setPresence({game: {name: "with my code", type: 0}});
});
```

```js
// Set the bot's online/idle/dnd/invisible status
client.on("ready", () => {
    client.user.setStatus("online");
});
```

## Users and Members

> In these examples `Guild` is a placeholder for where you get the guild. This can be `message.guild` or `member.guild` or just `guild` depending on the event. Or, you can get the guild by ID (see next section) and use that, too!

```js
// Get a User by ID
client.users.get("user id here");
// Returns <User>
```

```js
// Get a Member by ID
message.guild.members.get("user ID here");
// Returns <Member>
```

```js
// Get a Member from message Mention
message.mentions.members.first();
// Returns <Member>
```

```js
// Send a Direct Message to a user
message.author.send("hello");
// With Member it works too: 
```

```js
// Mention a user in a message
message.channel.send(`Hello ${user}, and welcome!`);
// or
message.channel.send("Hello " + message.author.toString() + ", and welcome!");
```

## Channels and Guilds

```js
// Get a Guild by ID
client.guilds.get("the guild id");
// Returns <Guild>
```

```js
// Get a channel by ID
client.channels.get("the channel id");
// Returns <TextChannel>
```

```js
// Get a Channel by Name (note: THIS IS NOT RECOMMENDED as more than one channel can have the same name!)
message.guild.channels.find("name", "channel-name");
// returns <TextChannel>
```

## Messages

```js
// Editing a message the bot sent
message.channel.send("Test").then(sentMessage => sentMessage.edit("Blah"));
// message now reads : "Blah"
```

```js
// Fetching a message by ID (Discord.js versions 9 through 11)
// note: you can line return right before a "dot" in JS, that is valid.
message.channel.fetchMessages({around: "352292052538753025", limit: 1})
  .then(messages => {
    const fetchedMsg = messages.first(); // messages is a collection!)
    // do something with it
    fetchedMsg.edit("This fetched message was edited");
  });

// THIS CHANGES IN DISCORD VERSION 12!!!!!
message.channel.messages.fetch({around: "352292052538753025", limit: 1})
  .then(messages => {
    messages.first().edit("This fetched message was edited");
  });
```
