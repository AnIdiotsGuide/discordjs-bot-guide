---
description: >-
  Some very quick code samples for common actions on the bot, users, members,
  channels and messages!
---

# Frequently Asked Questions

In this page, some very basic, frequently-asked questions are answered. It's important to understand that **these examples are generic** and will most likely not work if you just copy/paste them in your code. You need to **understand** these lines, not just blindly shove them in your code.

## Code Examples

## Bot and Bot Client

```javascript
// Set the bot's "Playing: " status (must be in an event!)
client.on("ready", () => {
    client.user.setActivity("my code", { type: "WATCHING"})
})
```

```javascript
// Set the bot's online/idle/dnd/invisible status
client.on("ready", () => {
    client.user.setStatus("online");
});
```

```javascript
// Set the bot's presence (activity and status)
client.on("ready", () => {
    client.user.setPresence({
        activities: [{ 
          name: "my code",
          type: "WATCHING"
        }],
        status: "idle"
    })
})
```

Note: You can find a list of all possible activity types [here](https://discord.js.org/#/docs/main/stable/typedef/ActivityType).

{% hint style="info" %}
If you want your bot's status to show up as `STREAMING`, you need to provide a Twitch or YouTube URL.
For `setActivity`, you need to provide an options object, which needs to have the URL, and the type should be set to streaming.
For `setPresence`, you need to provide the presence data object, which needs to contain the activities array, with the url and type \(Set it to "STREAMING"\).
{% endhint %}

```javascript
client.on("ready", () => {
    client.user.setActivity("my code", { type: "STREAMING", url: "https://www.twitch.tv/an_idiots_guide" })
})
```

## Users and Members

{% hint style="info" %}
In these examples `Guild` is a placeholder for where you get the guild. This can be `message.guild` or `member.guild` or just `guild` depending on the event. Or, you can get the guild by ID \(see next section\) and use that, too!
{% endhint %}

```javascript
// Get a User by ID
client.users.cache.get("user id here");
// Returns <User>
```

```javascript
// Get a Member by ID
message.guild.members.cache.get("user ID here");
// Returns <Member>
```

```javascript
// Get a Member from message Mention
message.mentions.members.first();
// Returns <Member>, if there is a mentioned member
```

```javascript
// Send a Direct Message to a user
message.author.send("hello");
// With Member it works too:
message.member.send("Heya!");
```

```javascript
// Mention a user in a message
message.channel.send(`Hello ${message.author.toString()}, and welcome!`);
// or
message.channel.send("Hello " + message.author.toString() + ", and welcome!");
```

```javascript
// Restrict a command to a specific user by ID
if (message.content.startsWith(`${prefix}commandname`)) {
    if (message.author.id !== "A user ID") return;
    // Your Command Here
}
```

```javascript
message.guild.members.fetch(message.author)
  .then(member => {
    // The member is available here.
  })
  .catch(() => {
    // When an error occurs.
  })
```

## Channels and Guilds

```javascript
// Get a Guild by ID
client.guilds.cache.get("the guild id");
// Returns <Guild>
```

```javascript
// Get a Channel by ID
client.channels.cache.get("the channel id");
// Returns <Channel>
```

```javascript
// Get a Channel by Name
message.guild.channels.cache.find(channel => channel.name === "channel-name");
// returns <Channel>
```

```javascript
// Create an invite and send it in the channel
// You can only create an invite from a GuildChannel
// Messages can only be sent to a TextChannel
message.guild.channels.cache.get('<CHANNEL ID>').createInvite().then(invite =>
    message.channel.send(invite.url)
);
```

### Default Channel

{% hint style="info" %}
As of 03/08/2017, **there is no more Default Channel** in guilds on Discord. The \#general default channel can be deleted, and the `guild.defaultChannel` property no longer works. As an alternative, for those _really_ wanting to send to what "looks" like the default channel, here's a dirty workaround.
{% endhint %}

Note: you'll need to `npm install long` and then `const Long = require("long");` to use the below code.

```javascript
const { Permissions } = require("discord.js");

const getDefaultChannel = (guild) => {
  // get "original" default channel
  if(guild.channels.cache.has(guild.id))
    return guild.channels.cache.get(guild.id)

  // Check for a "general" channel, which is often default chat
  const generalChannel = guild.channels.cache.find(channel => channel.name === "general");
  if (generalChannel)
    return generalChannel;
  // Now we get into the heavy stuff: first channel in order where the bot can speak
  // hold on to your hats!
  return guild.channels.cache
   .filter(c => c.type === "GUILD_TEXT" &&
     c.permissionsFor(guild.client.user).has(Permissions.FLAGS.SEND_MESSAGES))
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

It's very important to note that if the bot has admin perms, their "First writable channel" is the one on top. That could be Rules, Announcements, FAQs, whatever. So if the default channel was deleted and there's no general channel, you're going to annoy a lot of people.

Consider using [Enmap](https://npmjs.com/package/enmap) for per-guild settings instead \([example here](https://enmap.evie.dev/complete-examples/per-server-settings)\) and let server admins **choose** a channel!

## Messages

```javascript
// Editing a message the bot sent
message.channel.send("Test").then(sentMessage => sentMessage.edit("Blah"));
// message now reads : "Blah"
```

```javascript
message.channel.messages.fetch("352292052538753025")
  .then(message => {
    // do something with it
    // Check if the author of the message is the bot
    if (message.client.user.id !== message.id) return console.log("I'm not the author of that message!");
    // Edit the message
    message.edit("This fetched message was edited");
  });
```
