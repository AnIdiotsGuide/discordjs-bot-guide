# A Basic Command Handler Example

A _Command Handler_ is essentially a way to separate your commands into different files, instead of having a bunch of `if/else` conditions inside your code \(or a `switch/case` if you're being fancy\).

In this case, the code shows you how to separate each command into its own file. This means that each command can be _edited_ separately, and also _reloaded_ without the need to restart your bot. Yes, really!

> Want a better, updated version of this code? We're now maintaining this command handler at the community level. [Guide Bot is on Github](https://github.com/An-Idiots-Guide/guidebot/) and not only can you use the code, you can also contribute if you feel proficient enough!

## App.js Changes

Without going into all the details of the changes made, here is the modified app.js file:

```js
const Discord = require("discord.js");
const client = new Discord.Client();
const fs = require("fs");

const config = require("./config.json");

// This loop reads the /events/ folder and attaches each event file to the appropriate event.
fs.readdir("./events/", (err, files) => {
  if (err) return console.error(err);
  files.forEach(file => {
    let eventFunction = require(`./events/${file}`);
    let eventName = file.split(".")[0];
    // super-secret recipe to call events with all their proper arguments *after* the `client` var.
    client.on(eventName, (...args) => eventFunction.run(client, ...args));
  });
});

client.on("message", message => {
  if (message.author.bot) return;
  if(message.content.indexOf(config.prefix) !== 0) return;

  // This is the best way to define args. Trust me.
  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // The list of if/else is replaced with those simple 2 lines:
  try {
    let commandFile = require(`./commands/${command}.js`);
    commandFile.run(client, message, args);
  } catch (err) {
    console.error(err);
  }
});

client.login(config.token);
```

> **Fair Warning**: The require\(\) command here relies on _user input_ to call a file, which can be dangerous. Your homework is to find a way to make sure _only_ commands from the `./commands/` folder are called, and anything else such as `../../../../../etc/passwd` can't be loaded in memory because someone knows linux!

## Example commands

This would be the content of the `./commands/ping.js` file, which is called with `!ping` \(assuming `!` as a prefix\)

```js
exports.run = (client, message, args) => {
    message.channel.send("pong!").catch(console.error);
}
```

Another example would be the more complex `./commands/kick.js` command, called using `!kick @user`

```js
exports.run = (client, message, [mention, ...reason]) => {
  const modRole = message.guild.roles.find("name", "Mods");
  if (!modRole) 
    return console.log("The Mods role does not exist");
    
  if (!message.member.roles.has(modRole.id))
    return message.reply("You can't use this command.");

  if (message.mentions.users.size === 0) {
    return message.reply("Please mention a user to kick");

  if (!message.guild.me.hasPermission("KICK_MEMBERS"))
    return message.reply("");
    
  const kickMember = message.mentions.members.first();

  kickMember.kick(reason.join(" ")).then(member => {
    message.reply(`${member.user.username} was succesfully kicked.`);
  });
}
```

Notice the structure on the first line. `exports.run` is the "function name" that is exported, with 3 arguments: `client` \(the client\), `message` \(the message variable from the handler\) and `args`. Here, `args` is replaced by fancy destructuring that captures the `reason` (the rest of the message after the mention) in an array. See [Commands with Arguments](/examples/command_with_arguments.md) for details.

## Example Events

Events are handled almost exactly in the same way, except that the number of arguments depends on which event it is. For example, the `ready` event:

```js
exports.run = (client) => {
  console.log(`Ready to server in ${client.channels.size} channels on ${client.guilds.size} servers, for a total of ${client.users.size} users.`);
}
```

Note that the `ready` event normally doesn't have any arguments, it's just \(\). But because we're in separate modules, it's necessary to "pass" the `client` variable to it or it would not be accessible.

Here's another example with the `guildMemberAdd` event:

```js
exports.run = (client, member) => {
  let guild = member.guild;
  guild.defaultChannel.send(`Welcome ${member.user} to this server.`).catch(console.error);
}
```

Now we have `client` and also `member` which is the argument provided _by_ the `guildMemberAdd` event.

## BONUS: The "reload" command

Because of the way `require()` works in node, if you modify any of the command files in `./commands` , the changes are not reflected immediately when you call that command again - because `require()` _caches_ the file in memory instead of reading it every time. While this is great for efficiency, it means we need to clear that cached version if we change commands.

The _Reload_ command does just that, simply deletes the cache so the next time that specific command is run, it'll refresh its code from the file.

```js
exports.run = (client, message, args) => {
  if(!args || args.size < 1) return message.reply("Must provide a command name to reload.");
  // the path is relative to the *current folder*, so just ./filename.js
  delete require.cache[require.resolve(`./${args[0]}.js`)];
  message.reply(`The command ${args[0]} has been reloaded`);
};
```
