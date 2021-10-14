# A Basic Command Handler

A _Command Handler_ is essentially a way to separate your commands into different files, instead of having a bunch of `if/else` conditions inside your code \(or a `switch/case` if you're being fancy\).

In this case, the code shows you how to separate each command into its own file. This means that each command can be _edited_ separately, and also _reloaded_ without the need to restart your bot. Yes, really!

{% hint style="info" %}
Want a better, updated version of this code? We're now maintaining this command handler at the community level. [Guide Bot is on Github](https://github.com/AnIdiotsGuide/guidebot/) and not only can you use the code, you can also contribute if you feel proficient enough!
{% endhint %}

## What you need to know

In order to correctly write and use a command handler, I would suggest you get familiar with a few things.

* Check out [Introduction to Modules](https://js.evie.dev/modules) for information on modules \(which we'll use for each command\)
* Understand how [Events](../understanding/events-and-handlers.md) work, and how each event has different arguments provided.
* Have a good grasp of, at the very least, [Commands with Arguments](command-with-arguments.md), which we'll be using as a base for most of our code.

## Main File Changes

Because we're creating a separate file \(module\) for each event and each commands, our main file \(app.js, or index.js, or whatever you're calling it\) will change drastically from a list of commands to a simple file that loads other files.

Two main loops are needed to execute this master plan. First off, the one that will load all the `events` files. Each event will need to have a file in that folder, named _exactly_ like the event itself. So for `messageCreate` we want `./events/messageCreate.js`, for `guildBanAdd` we want `./events/guildBanAdd.js` , etc.

```javascript
// Read the Files in the Events Directory and filter files that ends with .js
const files = fs.readdirSync("./events").filter(file => file.endsWith(".js"));
// Loop over each file
for (const file of files) {
  // Split the file at its extension and get the event name
  const eventName = file.split(".")[0];
  // Require the file
  const event = require(`./events/${file}`);
    // super-secret recipe to call events with all their proper arguments *after* the `client` var.
    // without going into too many details, this means each event will be called with the client argument,
    // followed by its "normal" arguments, like message, member, etc etc.
    // This line is awesome by the way. Just sayin'.
  client.on(eventName, event.bind(null, client));
}
```

The second loop is going to be for the commands themselves. For a couple of reasons, we want to put the commands inside of a structure that we can refer to later - we'll use a Discord Collection:

```javascript
client.commands = new Discord.Collection();
// Read the Commands Directory, and filter the files that end with .js
const commands = fs.readdirSync("./commands").filter(file => file.endsWith(".js"));
// Loop over the Command files
for (const file of commands) {
  // Get the command name from splitting the file
  const commandName = file.split(".")[0];
  // Require the file
  const command = require(`./commands/${file}`);

  console.log(`Attempting to load command ${commandName}`);
  // Set the command to a collection
  client.commands.set(command.name, command);
}
```

Ok so with that being said, our main file now looks like this \(how _clean_ is that, really?\):

```javascript
const { Client, Intents } = require("discord.js");
const fs = require("fs");

const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
const config = require("./config.json");
// We also need to make sure we're attaching the config to the CLIENT so it's accessible everywhere!
client.config = config;
client.commands = new Discord.Collection();

const events = fs.readdirSync("./events").filter(file => file.endsWith(".js"));
for (const file of events) {
  const eventName = file.split(".")[0];
  const event = require(`./events/${file}`);
  client.on(eventName, event.bind(null, client));
}

const commands = fs.readdirSync("./commands").filter(file => file.endsWith(".js"));
for (const file of commands) {
  const commandName = file.split(".")[0];
  const command = require(`./commands/${file}`);

  console.log(`Attempting to load command ${commandName}`);
  client.commands.set(command.name, command);
}

client.login(config.token);
```

## Our first Event: Message

The `messageCreate` event is obviously the most important one, as it will receive all messages sent to the bot. Create the `./events/messageCreate.js` file \(make sure it's spelled _exactly_ like that\) and look at this bit of code:

```javascript
module.exports = (client, message) => {
  // Ignore all bots
  if (message.author.bot) return;

  // Ignore messages not starting with the prefix (in config.json)
  if (message.content.indexOf(client.config.prefix) !== 0) return;

  // Our standard argument/command name definition.
  const args = message.content.slice(client.config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Grab the command data from the client.commands Enmap
  const cmd = client.commands.get(command);

  // If that command doesn't exist, silently exit and do nothing
  if (!cmd) return;

  // Run the command
  cmd.run(client, message, args);
};
```

There are more things we could do here, like get per-guild settings or check permissions before running the command, etc. Out of a desire to keep this page simple, I've avoided all that extra code, but you can still find it on the [GuideBot repository](https://github.com/AnIdiotsGuide/guidebot/)!

## Example commands

This would be the content of the `./commands/ping.js` file, which is called with `!ping` \(assuming `!` as a prefix\)

```javascript
exports.run = (client, message, args) => {
    message.channel.send("pong!").catch(console.error);
}
```

Another example would be the more complex `./commands/kick.js` command, called using `!kick @user`

```javascript
exports.run = (client, message, [mention, ...reason]) => {
  const modRole = message.guild.roles.cache.find(role => role.name === "Mods");
  if (!modRole)
    return console.log("The Mods role does not exist");

  if (!message.member.roles.cache.has(modRole.id))
    return message.reply("You can't use this command.");

  if (message.mentions.members.size === 0)
    return message.reply("Please mention a user to kick");

  if (!message.guild.me.hasPermission("KICK_MEMBERS"))
    return message.reply("I don't have the `KICK_MEMBERS` permission");

  const kickMember = message.mentions.members.first();

  kickMember.kick(reason.join(" ")).then(member => {
    message.reply(`${member.user.username} was successfully kicked.`);
  });
};
```

Notice the structure on the first line. `exports.run` is the "function name" that is exported, with 3 arguments: `client` \(the client\), `message` \(the message variable from the handler\) and `args`. Here, `args` is replaced by fancy destructuring that captures the `reason` \(the rest of the message after the mention\) in an array. See [Commands with Arguments](command-with-arguments.md) for details.

## Other Events

Events are handled almost exactly in the same way, except that the number of arguments depends on which event it is. For example, the `ready` event:

```javascript
module.exports = (client) => {
  console.log(`Ready to serve in ${client.channels.cache.size} channels on ${client.guilds.cache.size} servers, for a total of ${client.users.cache.size} users.`);
}
```

Note that the `ready` event normally doesn't have any arguments, it's just `()`. But because we're in separate modules, it's necessary to "pass" the `client` variable to it or it would not be accessible. That's what our fancy `bind` is for in the main file!

Here's another example with the `guildMemberAdd` event:

```javascript
const { Permissions } = require("discord.js");

module.exports = (client, member) => {
  const defaultChannel = member.guild.channels.cache.find(channel => channel.permissionsFor(guild.me).has(Permissions.FLAGS.SEND_MESSAGES));
  defaultChannel.send(`Welcome ${member.user} to this server.`).catch(console.error);
}
```

Now we have `client` and also `member` which is the argument provided _by_ the `guildMemberAdd` event.

## BONUS: The "reload" command

Because of the way `require()` works in node, if you modify any of the command files in `./commands` , the changes are not reflected immediately when you call that command again - because `require()` _caches_ the file in memory instead of reading it every time. While this is great for efficiency, it means we need to clear that cached version if we change commands.

The _Reload_ command does just that, simply deletes the cache so the next time that specific command is run, it'll refresh its code from the file.

```javascript
exports.run = (client, message, args) => {
  if (!args || args.length < 1) return message.reply("Must provide a command name to reload.");
  const commandName = args[0];
  // Check if the command exists and is valid
  if (!client.commands.has(commandName)) {
    return message.reply("That command does not exist");
  }
  // the path is relative to the *current folder*, so just ./filename.js
  delete require.cache[require.resolve(`./${commandName}.js`)];
  // We also need to delete and reload the command from the client.commands Enmap
  client.commands.delete(commandName);
  const props = require(`./${commandName}.js`);
  client.commands.set(commandName, props);
  message.reply(`The command ${commandName} has been reloaded`);
};
```

Remember that all of this is just a fairly basic version of the GuideBot command handler which also has permissions, levels, per-guild configurations, and a whole lot of example commands and events! [Head on over to Github](https://github.com/AnIdiotsGuide/guidebot/) to see the completed handler.

Next up is making this basic command handler _better_ with the addition of [slash commands](slash-commands/md).
