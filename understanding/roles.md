# Roles and Permissions

Roles are a powerful feature in Discord, and admittedly have been one of the hardest parts to master in discord.js. This walk through aims at explaining how roles and permissions work. We'll also explore how to use roles to protect your commands.

## Role hierarchy

Let's start with a basic overview of the hierarchy of roles in Discord.

... or actually not, they already explain it better than I care to: [Role Management 101](https://support.discord.com/hc/en-us/articles/214836687-Role-Management-101). Read up on that, then come back here. I'll wait. \(Yeah I know that's cheesy, so sue me\).

## Role code

Let's get down to brass tacks. You want to know how to use roles and permissions in your bot.

### Get Role by Name or ID

This is the "easy" part once you actually get used to it. It's just like getting any other Collection element, but here's a reminder anyway!

```javascript
// get role by ID
let myRole = message.guild.roles.cache.get("264410914592129025");

// get role by name
let myRole = message.guild.roles.cache.find(role => role.name === "Moderators");
```

{% hint style="info" %}
To get the ID of a role, you can either mention it with a `\` before it, like `\@rolename`, or copy it from the role menu. If you mention it, the ID is the numbers between the `<>`. To get the ID of a role without mentioning it, enable developer mode in the Appearance section of your user settings, then go to the role menu in the server settings and right click on the role you want the ID of, then click "Copy ID".
{% endhint %}

### Check if a member has a role

In a `messageCreate` handler, you have access to checking the GuildMember class of the message author:

```javascript
// assuming role.id is an actual ID of a valid role:
if (message.member.roles.cache.has(role.id)) {
  console.log("Yay, the author of the message has the role!");
}

else {
  console.log("Nope, noppers, nadda.");
}
```

```javascript
// Check if they have one of many roles
if (message.member.roles.cache.some(r=>["Dev", "Mod", "Server Staff", "Proficient"].includes(r.name)) ) {
  // has one of the roles
}

else {
  // has none of the roles
}
```

{% hint style="info" %}
To grab members and users in different ways see the [FAQ Page](../frequently-asked-questions.md).
{% endhint %}

### Get all members that have a role

```javascript
let roleID = "264410914592129025";
let membersWithRole = message.guild.roles.cache.get(roleID).members;
console.log(`Got ${membersWithRole.size} members with that role.`);
```

### Add a member to a role

Alright, now that you have roles, you probably want to add a member to a role. Simple enough! Discord.js provides 2 handy methods to add, and remove, a role. Let's look at them!

```javascript
let role = message.guild.roles.cache.find(r => r.name === "Team Mystic");

// Let's pretend you mentioned the user you want to add a role to (!addrole @user Role Name):
let member = message.mentions.members.first();

// or the person who made the command: let member = message.member;

// Add the role!
member.roles.add(role).catch(console.error);

// Remove a role!
member.roles.remove(role).catch(console.error);
```

Alright I feel like I have to add a _little_ precision here on implementation:

* You can **not** add or remove a role that is higher than the bot's. This should be obvious.
* The bot requires `MANAGE_ROLES` permissions for this. You can check for it using the code further down this page.
* Because of global rate limits, you cannot do 2 role "actions" immediately one after the other. The first action will work, the second will not. You can go around that by using `<GuildMember>.roles.set([array, of, roles])`. This will overwrite all existing roles and only apply the ones in the array so be careful with it.

## Permission code

### Check specific permission of a member on a channel

To check for a single permission override on a channel:

```javascript
// Getting all permissions for a member on a channel.
let perms = message.channel.permissionsFor(message.member);

// Checks for Manage Messages permissions.
let can_manage_messages = message.channel.permissionsFor(message.member).has("MANAGE_MESSAGES", false);

// View permissions as an object (useful for debugging or eval)
message.channel.permissionsFor(message.member).serialize(false)
```

{% hint style="info" %} We pass `false` for the checkAdmin parameter because Administrator channel overwrites don't implicitly grant any permissions, unlike in Roles or when you are the Guild Owner. \(The API will allow you to create an overwrite with Administrator, and even tell D.JS that a channel overwrite has had Administrator permissions set. Discord developers have stated this is [intended behavior](https://github.com/discord/discord-api-docs/issues/640).\)
{% endhint %}

### Get all permissions of a member on a guild

Just as easy, woah!

```javascript
let perms = message.member.permissions;

// Check if a member has a specific permission on the guild!
let has_kick = perms.has("KICK_MEMBERS");
```

ezpz, right?

Now get to coding!

## ADDENDUM: Permission Names

Click [here](https://discord.js.org/#/docs/main/master/class/Permissions?scrollTo=s-FLAGS) for the full list of internal permission names, used for `.has(name)` in the above examples
