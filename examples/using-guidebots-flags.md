---
description: >-
  This page is an explanation of how to use switch cases with a feature in
  Guidebot, message.flags
---

# Using Guidebot's Flags

## An Example

Switch cases are a nice feature that can be used without the `message.flags` feature, and can be used like this.

```javascript
// This will look for the first argument. If your command was 'hi', and you did 'hi send', it would send 'Hi!' to the channel.

switch (args[0]) {  
  case 'send': {
    // This will send in the channel that the command was run in.
    message.channel.send('Hi!');
    break;
  }

  case 'dm': {
    // This will DM the message author..
    message.author.send('Hi!');
  }
}
```

However, you can use the switch case with `message.flags`.

```javascript
// This will look for the first argument beginning with -, the flag. If your command was 'hi', and you did 'hi -send', it would send 'Hi!' to the channel.
switch (message.flags[0]) {
  case 'send': {
    // This will send in the channel that the command was run in.
    message.channel.send('Hi!');
    break;
  }

  case 'dm': {
    // This will DM the message author.
    message.author.send('Hi!');
  }
}
```

## Using with Arguments

Let's say you're trying to make a command to add or take a role to/from a user. This can be consolidated into one command using `message.flags`. This command is based off of the [Guidebot](https://github.com/AnIdiotsGuide/guidebot) framework, and is called role.

```javascript
exports.run = async (client, message, args, level) => { // eslint-disable-line no-unused-vars 
  switch (message.flags[0]) {
    // This would be 'role -add'.
    case 'add': {
      // Check if the message mentions a user.
      if (message.mentions.members.size === 0) return message.reply('Please mention a user to give the role to.');
      const member = message.mentions.members.first();
      // This is the name of the role. For example, if you do 'role -add @York#2400 The Idiot Himself', the name of the role would be 'The Idiot Himself'.
      const name = args.slice(1).join(' ');
      // Find the role on the guild.
      const role = message.guild.roles.find('name', name);
      // End the command if the bot cannot find the role on the server.
      if (!role) return message.reply('I can\'t seem to find that role.');
      try {
        await member.addRole(role);
        await message.channel.send(`I've added the ${name} role to ${member.dsiplayName}.`);
      } catch (e) {
        console.log(e);
      }
      break;
    }

    case 'remove': {
      // Check if the message mentions a user.
      if (message.mentions.members.size === 0) return message.reply('Please mention a user to take the role from.');
      const member = message.mentions.members.first();
      // This is the name of the role. For example, if you do 'role -remove @York#2400 The Idiot Himself', the name of the role would be 'The Idiot Himself'.
      const name = args.slice(1).join(' ');
      // Find the role on the guild.
      const role = message.guild.roles.find('name', name);
      // End the command if the bot cannot find the role on the server.
      if (!role) return message.reply('I can\'t seem to find that role.');
      try {
        await member.removeRole(role);
        await message.channel.send(`I've removed the ${name} role from ${member.displayName}.`);
      } catch (e) {
        console.log(e);
      }
    }
  }
};

exports.conf = {
  enabled: true,
  guildOnly: true,
  aliases: [],
  permLevel: 'Moderator'
};

exports.help = {
  name: 'role',
  category: 'Moderation',
  description: 'It can give or take a role to/from a user.',
  usage: 'role <-add|-remove> <role name>'
};
```

