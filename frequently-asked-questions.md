---
description: >-
  Commonly asked questions about discord.js including things about members,
  channels, roles, etc.
---

# Frequently Asked Questions

In this page, some very basic, frequently-asked questions are answered. It's important to understand that **these examples are generic** and will most likely not work if you just copy/paste them in your code. You need to **understand** these lines, not just blindly shove them in your code.

## Code Examples

## Bot and Bot Client

```javascript
// Set the bot's "Playing: " status (must be in an event!)
client.on("ready", () => {
    client.user.setActivity({game: {name: "with my code", type: 0}});
});
```

```javascript
// Set the bot's online/idle/dnd/invisible status
client.on("ready", () => {
    client.user.setStatus("online");
});
```

## Users and Members

{% hint style="info" %}
In these examples `Guild` is a placeholder for where you get the guild. This can be `message.guild` or `member.guild` or just `guild` depending on the event. Or, you can get the guild by ID \(see next section\) and use that, too!
{% endhint %}

```javascript
// Get a User by ID
client.users.get("user id here");
// Returns <User>
```

```javascript
// Get a Member by ID
message.guild.members.get("user ID here");
// Returns <Member>
```

```javascript
// Get a Member from message Mention
message.mentions.members.first();
// Returns <Member>
```

```javascript
// Send a Direct Message to a user
message.author.send("hello");
// With Member it works too:
message.member.send("Heya!");
```

```javascript
// Mention a user in a message
message.channel.send(`Hello ${user}, and welcome!`);
// or
message.channel.send("Hello " + message.author.toString() + ", and welcome!");
```

```javascript
// Restrict a command to a specific user by ID
if (message.content.startsWith(prefix + 'commandname')) {
    if (message.author.id !== 'A user ID') return;
    // Your Command Here
}
```

```javascript
// FETCH a member. Useful if an invisible user sends a message.
message.guild.fetchMember(message.author)
  .then(member => {
    // The member is available here.
  });

// THIS CHANGES IN DISCORD VERSION 12!!!!!
message.guild.members.fetch(message.author)
  .then(member => {
    // The member is available here.
  });
```

## Channels and Guilds

```javascript
// Get a Guild by ID
client.guilds.get("the guild id");
// Returns <Guild>
```

```javascript
// Get a Channel by ID
client.channels.get("the channel id");
// Returns <Channel>
```

```javascript
// Get a Channel by Name
message.guild.channels.find("name", "channel-name");
// returns <Channel>
```

```javascript
// Create an invite and send it in the channel
// You can only create an invite from a GuildChannel
// Messages can only be sent to a TextChannel
message.guild.channels.get('<CHANNEL ID>').createInvite().then(invite =>
    message.channel.send(invite.url)
);
```

### Default Channel

{% hint style="info" %}
As of 03/08/2017, **there is no more Default Channel** in guilds on Discord. The \#general default channel can be deleted, and the `guild.defaultChannel` property no longer works. As an alternative, for those _really_ wanting to send to what "looks" like the default channel, here's a dirty workaround.
{% endhint %}

Note: you'll need to `npm install long` and then `var Long = require("long");` to use the below code.

```javascript
const getDefaultChannel = async (guild) => {
  // get "original" default channel
  if(guild.channels.has(guild.id))
    return guild.channels.get(guild.id)

  // Check for a "general" channel, which is often default chat
  if(guild.channels.exists("name", "general"))
    return guild.channels.find("name", "general");
  // Now we get into the heavy stuff: first channel in order where the bot can speak
  // hold on to your hats!
  return guild.channels
   .filter(c => c.type === "text" &&
     c.permissionsFor(guild.client.user).has("SEND_MESSAGES"))
   .sort((a, b) => a.position - b.position ||
     Long.fromString(a.id).sub(Long.fromString(b.id)).toNumber())
   .first();
}

// This is called as, for instance:
client.on("guildMemberAdd", member => {
  const channel = getDefaultChannel(member.guild);
  channel.send(`Welcome ${member} to the server, wooh!`);
});
```

It's very important to note that if the bot has admin perms, their "First writeable channel" is the one on top. That could be Rules, Announcements, FAQs, whatever. So if the default channel was deleted and there's no general channel, you're going to annoy a lot of people.

Consider using [Enmap](https://npmjs.com/package/enmap) for per-guild settings instead \([example here](https://gist.github.com/eslachance/5c539ccebde9fa76340fb5d54889aa22)\) and let server admins **choose** a channel!

## Messages

```javascript
// Editing a message the bot sent
message.channel.send("Test").then(sentMessage => sentMessage.edit("Blah"));
// message now reads : "Blah"
```

```javascript
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

