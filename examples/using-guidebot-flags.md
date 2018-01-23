# Using Switch Cases with Guidebot

This page is an explanation of how to use switch cases with a feature in Guidebot, `message.flags`.

## An Example

Switch cases are a nice feature that can be used without the `message.flags` feature, and can be used like this. 

```js
// This will look for the first argument. If your command was 'hi', and you did 'hi send', it would send 'Hi!' to the channel.
const trigger = args[0];

switch (trigger) {  
  case ('send'): {
    // This will send in the channel that the command was run in.
    message.channel.send('Hi!');
    break;
  }

  case ('dm'): {
    // This will DM the message author..
    message.author.send('Hi!');
  }
}
```

However, you can use the switch case with the neat little `message.flags` concept.

```js
  // This will look for the first argument beginning with -, the flag. If your command was 'hi', and you did 'hi -send', it would send 'Hi!' to the channel.
  switch (message.flags[0]) {
    case ('send'): {
    // This will send in the channel that the command was run in.
    message.channel.send('Hi!');
    break;
  }

  case ('dm'): {
    // This will DM the message author.
    message.author.send('Hi!');
  }
}
```

## Using with Arguments

Let's say you're trying to make a command to add or take a role to/from a user. This can be consolidated into one command using the `message.flags` method. This command is based off of the [Guidebot](https://github.com/AnIdiotsGuide/guidebot) framework, and is called role.

```js
exports.run = (client, message, args, level) => { // eslint-disable-line no-unused-vars 
  switch (message.flags[0]) {
    // This would be 'role -add'.
    case ('add'): {
      // Check if the message mentions a user.
      if (message.mentions.users.size === 0) return message.reply('Please mention a user to give the role to.');
      const member = message.guild.member(message.mentions.users.first());
      // This is the name of the role. F`or example, if you do 'role -add @York#2400 The Idiot Himself', the name of the role would be 'The Idiot Himself'.
      const name = args.slice(1).join(' ');
      // Find the role on the guild.
      const role = message.guild.roles.find('name', name);
      // End the command if the bot cannot find the role on the server.
      if (!role) return message.reply('I can't seem to find that role.');
      member.addRole(role).catch(e => {
        return message.channel.send(`Something went wrong! **Error:**\n${e}`);
      });
      message.channel.send(`I've added the ${name} role to ${message.mentions.users.first().username}.`);
      break;
    }

    case ('remove'): {
    // Check if the message mentions a user.
      if (message.mentions.users.size === 0) return message.reply('Please mention a user to take the role from.');
      const member = message.guild.member(message.mentions.users.first());
      // This is the name of the role. For example, if you do 'role -remove @York#2400 The Idiot Himself', the name of the role would be 'The Idiot Himself'.
      const name = args.slice(1).join(' ');
      // Find the role on the guild.
      const role = message.guild.roles.find('name', name);
      // End the command if the bot cannot find the role on the server.
      if (!role) return message.reply('I can't seem to find that role.');
      member.removeRole(role).catch(e => {
        return message.channel.send(`Something went wrong! **Error:**\n${e}`);
      });
      message.channel.send(`I've removed the ${name} role from ${message.mentions.users.first().username}.`);
      break;
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
  usage: 'ping'
};
  ```
}
