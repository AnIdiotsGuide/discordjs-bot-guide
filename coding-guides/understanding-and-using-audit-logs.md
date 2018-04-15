
# Understanding Audit Logs

From time to time, users in "An Idiot's Guide Official Server" need to reference audit logs for whatever reason;
let it be viewing them for certain actions to adding things into the audit logs. 
This guide will explain everything about audit logs and how to use them.

The first thing that you will need is a working Discord Bot. If you do not have one, please visit [Your First Bot](getting-started/your-basic-bot.md) to get started.

Now, some things to take note of. The bot will need some permissions. The main permission the bot will need
is 'VIEW_AUDIT_LOGs'. This permission allows the bot to view the audit logs and do whatever you want to, obeying the Discord ToS that is. 

Now that the permission has been established. Lets get started!

Firstly, we need to know what we are doing with the audit logs. 
Lets log who deleted a message using the messageDeleted event. This event will fire whenever a cached message is deleted in a server.

```js
client.on('messageDeleted', async (message) => {
  // Firstly, we need a logs channel. 
  const logs = message.guild.channels.find('name', 'logs');
  
  // If there is no logs channel, we can create it if we have the 'MANAGE_CHANNELS' permission
  // Remember, this is completely options. Use to your best judgement.
  if (message.guild.me.hasPermission('MANAGE_CHANNELS') && !logs) {
    message.guild.createChannel('logs', 'text');
  }
  
  // If we do not have permissions, console log both errors
  if (!message.guild.me.hasPermission('MANAGE_CHANNELS') && !logs) { 
    console.log('The logs channel does not exist and tried to create the channel but I am lacking permissions')
  }
  
  // Now that we have the logs channel created...hopefully...we can send messages to it
  // before we do that, lets establish who deleted the message
  // The "type" is how you will be searching through the audit logs. Like roles updated or members banned.
  const executor = await message.guild.fetchAuditLogs({type: 'MESSAGE_DELETE'}).then(audit => audit.entries.first())
  
  
})
```
Lets take a break to explain exactly whats going on. The executor will be retrieving the very latest audit log entry and all of its information that goes with it. What does this information contain? Everything we need for logging. 

```js
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
Notice the `reason` field. Some audit logs, like kicking and banning, can provide a reason. You can probably make logs of when a user is banned and for whatever reason.
What we want is the executor of the action. We do that by going to the `executor` target as that is where the user is stored.
So, our `executor` should be look similar to this:

```js
  const executor = await message.guild.fetchAuditLogs({type: 'MESSAGE_DELETE'}).then(audit => audit.entries.first().executor.username)
```

Wait wait! Where did the `.username` come from?
Excellent question! When you get the executor's information, it is considered a user.
We can get information about that executor, like their id, avatar, and more. 

```js
User {
  id: '286270602820452353',
  username: 'Zoth The Wumpus',
  discriminator: '6066',
  avatar: '57361ef0f8e23e02a44069c3dd5f5f47',
  bot: false,
  lastMessageID: '435165896198062091',
  lastMessage: [Object] }
```

So now that we have the following:
```js
// fetching audit logs is a promise, so we want to async the event
client.on('messageDeleted', async (message) => {
  // Firstly, we need a logs channel. 
  const logs = message.guild.channels.find('name', 'logs');
  
  // If there is no logs channel, we can create it if we have the 'MANAGE_CHANNELS' permission
  if (message.guild.me.hasPermission('MANAGE_CHANNELS') && !logs) {
    message.guild.createChannel('logs', 'text');
  }
  
  // If we do not have permissions, console log both errors
  if (!message.guild.me.hasPermission('MANAGE_CHANNELS') && !logs) { 
    console.log('The logs channel does not exist and tried to create the channel but am lacking permissions')
  }
  
  // Now that we have the logs channel created...hopefully...we can send messages to it
  // before we do that, lets get started to viewing the audit logs. 
  // We want the user who deleted the message
  // The "type" is how you will be searching through the audit logs. Like roles updated or members banned.
  const executor = await message.guild.fetchAuditLogs({type: 'MESSAGE_DELETE'}).then(audit => audit.entries.first().executor.username)
})
```
lets start sending it all to a channel.

```js
  // I always format my messages in strings as it's easier (for me) to understand what I am doing as some strings can get pretty long,
  // like the $5 foot long from subway. Man I miss that. 
  const deletedMessageInformation = "A message was deleted in " + message.channel.name + " by " + executor;
  
  // Now lets send the message to the channel
  logs.send(deletedMessageInformation);
```

And there you have it. Thats how you can view audit logs as most of them, if not all, of them work the same. 

Types of Audit Logs:
```js
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
```
