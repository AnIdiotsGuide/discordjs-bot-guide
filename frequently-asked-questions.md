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
