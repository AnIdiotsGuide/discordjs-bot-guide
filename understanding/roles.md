---
description: >-
  Let's explore how to get, use, assign and list roles, as well as how to work
  with permissions.
---

# Roles and Permissions

Roles are a powerful feature in Discord, and admittedly have been one of the hardest parts to master in discord.js. This walkthrough aims at explaining how roles and permissions work. We'll also explore how to use roles to protect your commands.

## Role hierarchy

Let's start with a basic overview of the hierarchy of roles in Discord.

... or actually not, they already explain it better than I care to: [Role Management 101](https://support.discordapp.com/hc/en-us/articles/214836687-Role-Management-101). Read up on that, then come back here. I'll wait. \(Yeah I know that's cheesy, so sue me\).

## Role code

Let's get down to brass tacks. You want to know how to use roles and permissions in your bot.

### Get Role by Name or ID

This is the "easy" part once you actually get used to it. It's just like getting any other Collection element, but here's a reminder anyway!

```javascript
// get role by ID
let myRole = message.guild.roles.get("264410914592129025");

// get role by name
let myRole = message.guild.roles.find("name", "Moderators");
```

{% hint style="info" %}
To get a role ID, mention the role with a `\` before it in a Discord channel \(e.g. `\@rolename`\). You can then grab the ID from the output message. Right-clicking a role does nothing in the UI, there's currently no way to get it from there.
{% endhint %}

### Check if a member has a role

In a `message` handler, you have access to checking the GuildMember class of the message author:

```javascript
// assuming role.id is an actual ID of a valid role:
if(message.member.roles.has(role.id)) {
  console.log(`Yay, the author of the message has the role!`);
} else {
  console.log(`Nope, noppers, nadda.`);
}
```

```javascript
// Check if they have one of many roles
if(message.member.roles.some(r=>["Dev", "Mod", "Server Staff", "Proficient"].includes(r.name)) ) {
  // has one of the roles
} else {
  // has none of the roles
}
```

{% hint style="info" %}
To grab members and users in different ways see the [FAQ Page](../frequently-asked-questions.md).
{% endhint %}

### Get all members that have a role

```javascript
let roleID = "264410914592129025";
let membersWithRole = message.guild.roles.get(roleID).members;
console.log(`Got ${membersWithRole.size} members with that role.`);
```

### Add a member to a role

Alright, now that you have roles, you probably want to add a member to a role. Simple enough! Discord.js provides 2 handy methods to add, and remove, a role. Let's look at them!

```javascript
let role = message.guild.roles.find("name", "Team Mystic");

// Let's pretend you mentioned the user you want to add a role to (!addrole @user Role Name):
let member = message.mentions.members.first();

// or the person who made the command: let member = message.member;

// Add the role!
member.addRole(role).catch(console.error);

// Remove a role!
member.removeRole(role).catch(console.error);
```

Alright I feel like I have to add a _little_ precision here on implementation:

* You can **not** add or remove a role that is higher than the bot's. This should be obvious.
* The bot requires `MANAGE_ROLES` permissions for this. You can check for it using the code further down this page.
* Because of global rate limits, you cannot do 2 role "actions" immediately one after the other. The first action will work, the second will not. You can go around that by using `<GuildMember>.setRoles([array, of, roles])`. This will overwrite all existing roles and only apply the ones in the array so be careful with it.

## Permission code

### Check specific permission of a member on a channel

To check for a single permission override on a channel:

```javascript
// Getting all permissions for a member on a channel.
let perms = message.channel.permissionsFor(message.member);

// Checks for Manage Messages permissions.
let can_manage_chans = message.channel.permissionsFor(message.member).has("MANAGE_MESSAGES", false);

// View permissions as an object (useful for debugging or eval)
message.channel.permissionsFor(message.member).serialize(false)
```

> Note: We pass `false` for the checkAdmin parameter because Administrator channel overwrites don't implicently grant any permissions, unlike in Roles or when you are the Guild Owner. \(The API will allow you to create an overwrite with Administrator, and even tell D.JS that a channel overwrite has had Administrator permissions set. Discord devs have stated this is [intended behavior](https://github.com/discordapp/discord-api-docs/issues/640).\)

### Get all permissions of a member on a guild

Just as easy, wooh!

```javascript
let perms = message.member.permissions;

// Check if a member has a specific permission on the guild!
let has_kick = perms.has("KICK_MEMBERS");
```

ezpz, right?

Now get to coding!

## ADDENDUM: Permission Names

This is the list of internal permission names, used for `.has(name)` in the above examples:

```javascript
{
  CREATE_INSTANT_INVITE: true,
  KICK_MEMBERS: true,
  BAN_MEMBERS: true,
  ADMINISTRATOR: true,
  MANAGE_CHANNELS: true,
  MANAGE_GUILD: true,
  ADD_REACTIONS: true,
  READ_MESSAGES: true,
  SEND_MESSAGES: true,
  SEND_TTS_MESSAGES: true,
  MANAGE_MESSAGES: true,
  EMBED_LINKS: true,
  ATTACH_FILES: true,
  READ_MESSAGE_HISTORY: true,
  MENTION_EVERYONE: true,
  EXTERNAL_EMOJIS: true,
  CONNECT: true,
  SPEAK: true,
  MUTE_MEMBERS: true,
  DEAFEN_MEMBERS: true,
  MOVE_MEMBERS: true,
  USE_VAD: true,
  CHANGE_NICKNAME: true,
  MANAGE_NICKNAMES: true,
  MANAGE_ROLES_OR_PERMISSIONS: true,
  MANAGE_WEBHOOKS: true,
  MANAGE_EMOJIS: true
}
```

