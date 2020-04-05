# Using Audit Logs

From time to time, users in "An Idiot's Guide Official Server" need to reference audit logs for whatever reason, let it be viewing them for certain actions to adding things into the audit logs. This guide will explain everything about audit logs and how to use them.

The first thing that you will need is a working Discord Bot. If you do not have one, please visit [Your First Bot](../first-bot/your-first-bot.md) to get started. Now, some things to take note of. This guide is using discord.js@11.3.2. The bot will need some permissions. The main permission the bot will need is `'VIEW_AUDIT_LOGS'`. This permission allows the bot to view the audit logs.

Now that the permission has been established. Lets get started!

Firstly, we need to know what we are doing with the audit logs. Let's log who deleted a message using the messageDelete event. This event will fire whenever a cached message is deleted in a server.

```javascript
client.on('messageDelete', async (message) => {
  // Firstly, we need a logs channel. 
  const logs = message.guild.channels.find(channel => channel.name === "logs");

  // If there is no logs channel, we can create it if we have the 'MANAGE_CHANNELS' permission
  // Remember, this is completely options. Use to your best judgement.
  if (message.guild.me.hasPermission('MANAGE_CHANNELS') && !logs) {
    await message.guild.createChannel('logs', 'text');
  }

  // If we do not have permissions, console log both errors
  if (!logs) { 
    return console.log('The logs channel does not exist and cannot be created');
  }

  /*
  Lets establish who deleted the message. This is the actual audit logs part, yay!
  The "type" is how you will be searching through the audit logs, like role 
  updates or members banned. A complete list of types can be found at the end of this page.
  Keep in mind the following line uses some advanced async/await promise manipulation. 
  Explaining exactly how this works is beyond the scope of this guide.
  */
  const entry = await message.guild.fetchAuditLogs({type: 'MESSAGE_DELETE'}).then(audit => audit.entries.first())

  // Define an empty user for now. This will be used later in the guide.
  let user;

})
```

This is what is returned from `entry`. This information allows us to compare the time stamp, the user, the channel, and the executor.

```javascript
GuildAuditLogsEntry {
  targetType: 'MESSAGE',
  actionType: 'DELETE',
  action: 'MESSAGE_DELETE',
  reason: null,
  executor: [Object],
  changes: null,
  id: '434116792567201792',
  extra: [Object],
  target: [Object] }
```

Notice the `reason` field. Some audit logs, like kicking and banning, can provide a reason. You can probably make the bot log when a user is banned and for whatever reason. What we want is the executor of the action. We do that by going to the `executor` target as that is where the user is stored. Below is showing what the information that the `executor` property provides us:

```javascript
User {
  id: '286270602820452353',
  username: 'Zoth The Wumpus',
  discriminator: '6066',
  avatar: '57361ef0f8e23e02a44069c3dd5f5f47',
  bot: false,
  lastMessageID: '435165896198062091',
  lastMessage: [Object] }
```

With all the information above, we can start creating comparisons to narrow down on who really deleted the message, whether it was the author of the message or someone else.

```javascript
// Please keep in mind: Discord's audit logs will not log the information if the author of that message deleted it.
// I did this with a series of checks:
​ 
//we defined entry above, so we can use it here to check the channel id
if (entry.extra.channel.id === message.channel.id
​ 
//Then we are checking if the target is the same as the author id
&& (entry.target.id === message.author.id)
​ 
// We are comparing time as audit logs are sometimes slow. 
&& (entry.createdTimestamp > (Date.now() - 5000)

// We want to check the count as audit logs stores the amount deleted in a channel
&& entry.extra.count >= 1) {
  user = entry.executor.username
} else { 
  // When all else fails, we can assume that the deleter is the author.
  user = message.author.username
}
```

Let's take a break to explain exactly whats going on in the above code block. The `Date.now()` is getting the current time \(in milliseconds\). We want to take away 5 seconds for the potential delay in the audit logs. The `entry` will be retrieving the very latest audit log entry and all of its information that goes with it. What does this information contain? Everything we need for logging. With all the given information above, let's start sending it all to a channel.

```javascript
  // We defined the logs channel earlier in this guide, so now we can send it to the channel!
  logs.send(`A message was deleted in ${message.channel.name} by ${user}`;);
```

The final code should look like this:

```javascript
client.on('messageDelete', async (message) => {
  const logs = message.guild.channels.find(channel => channel.name === "logs");
  if (message.guild.me.hasPermission('MANAGE_CHANNELS') && !logs) {
    message.guild.createChannel('logs', 'text');
  }
  if (!message.guild.me.hasPermission('MANAGE_CHANNELS') && !logs) { 
    console.log('The logs channel does not exist and tried to create the channel but I am lacking permissions')
  }  
  const entry = await message.guild.fetchAuditLogs({type: 'MESSAGE_DELETE'}).then(audit => audit.entries.first())
  let user = ""
    if (entry.extra.channel.id === message.channel.id
      && (entry.target.id === message.author.id)
      && (entry.createdTimestamp > (Date.now() - 5000))
      && (entry.extra.count >= 1)) {
    user = entry.executor.username
  } else { 
    user = message.author.username
  }
  logs.send(`A message was deleted in ${message.channel.name} by ${user}`);
})
```

And there you have it. Thats how you can view audit logs as most of them, if not all, of them work the same.

Types of Audit Logs:

```javascript
GUILD_UPDATE
CHANNEL_CREATE
CHANNEL_UPDATE
CHANNEL_DELETE
CHANNEL_OVERWRITE_CREATE
CHANNEL_OVERWRITE_UPDATE
CHANNEL_OVERWRITE_DELETE
MEMBER_KICK
MEMBER_PRUNE
MEMBER_BAN_ADD
MEMBER_BAN_REMOVE
MEMBER_UPDATE
MEMBER_ROLE_UPDATE
MEMBER_MOVE
MEMBER_DISCONNECT
BOT_ADD
ROLE_CREATE
ROLE_UPDATE
ROLE_DELETE
INVITE_CREATE
INVITE_UPDATE
INVITE_DELETE
WEBHOOK_CREATE
WEBHOOK_UPDATE
WEBHOOK_DELETE
EMOJI_CREATE
EMOJI_UPDATE
EMOJI_DELETE
MESSAGE_DELETE
MESSAGE_BULK_DELETE
MESSAGE_PIN
MESSAGE_UNPIN
INTEGRATION_CREATE
INTEGRATION_UPDATE
INTEGRATION_DELETE
```

