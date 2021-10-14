# SQLite-Based Points System

That is the focus of this guide: we'll be creating the points system with SQLite. The core of this system is using the `better-sqlite3` package that you can get from [npmjs.com](https://npmjs.com/package/better-sqlite3).

## Installation

{% hint style="warning" %}
**Pre-Requisites**: `better-sqlite3`, similarly to a lot of modules, gets compiled using `node-gyp` which has 2 very important requirements: Python 2.7 and the C++ Build Tools. For windows, open up an Elevated \(Administrator\) command prompt and run the following FIRST, before installing better-sqlite3: `npm i --vs2015 -g windows-build-tools`. For linux, you need `sudo apt-get install build-essential` and you need to figure out how to install Python 2.7 \(NOT Python 3!\) on your system.
{% endhint %}

For this guide to work, you first need to make sure you have the proper modules installed. Let's assume you already have `discord.js` installed, and go straight to installing the sqlite one and its node-gyp dependency:

```text
npm i node-gyp better-sqlite3
```

## Setting the table

You've got SQLite installed, now what do we want in our table? I'll tell ya!

For this example points system we want the user's ID, points and level to be compliant with Discord's Developer Terms of Service. I'm not going to go to deep in to the jargon surrounding SQL and SQLite, but the tables are made from rows and columns of data. Got it? Good, moving on!

Our starting point is a very basic message handler with pre-existing commands - such as what we see in the [Command with Arguments](../first-bot/command-with-arguments.md) page of this guide. The code is as such:

```javascript
const { Client, Intents } = require("discord.js");
const client = new Client({
  intents: [Intents.FLAGS.GUILDS, Intents.FLAGS.GUILD_MESSAGES]
});
const config = require("./config.json"); // Contains the prefix, and token!

client.on("ready", () => {
  console.log("I am ready!");
});

client.on("messageCreate", message => {
  if (message.author.bot) return;
  // This is where we'll put our code.
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Command-specific code here!
});

client.login(config.token);
```

Now we've got that we should `require` sqlite and make use of it, put the following under `const client`:

```javascript
const SQLite = require("better-sqlite3");
const sql = new SQLite("./scores.sqlite");
```

We do have a small caveat - we really don't want to react on Direct Messages, so our whole code will be in a block that checks for that. We don't just want to ignore DMs because our bot itself might have DM commands!

```javascript
client.on("messageCreate", message => {
  if (message.author.bot) return;
  if (message.guild) {
    // This is where we'll put our code.
  }
  // Rest of message handler
});
```

Your code should look like this now:

```javascript
const { Client } = require("discord.js");
const client = new Client();
const config = require("./config.json");
const SQLite = require("better-sqlite3");
const sql = new SQLite("./scores.sqlite");

client.on("ready", () => {
  console.log("I am ready!");
});

client.on("messageCreate", message => {
  if (message.author.bot) return;
    if (message.guild) {
    // This is where we'll put our code.
  }
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Command-specific code here!
});

client.login(config.token);
```

## Dressing the Table

One important thing with SQLite is that it will only create tables if we ask it to. That means, we have to make sure that the table exists. We'll do this in our `ready` event, so it will execute only once when we start the bot. Now, I'm doing _a little bit of magic_ here, which includes setting some database toggles that make SQLite faster. If you want to look up "pragma syncronous" and "pragma journal mode wal", by all means go learn what they are, but these are good _production_ settings to have.

And, another bit of magic here, is that we can _prepare_ some statements beforehand, and simply _run_ them with specific values later on. This is a more advanced concept of SQL, but it should be easy to follow even if you're not familiar with it. So here's the code for the ready event:

```javascript
client.on("ready", () => {
  // Check if the table "points" exists.
  const table = sql.prepare("SELECT count(*) FROM sqlite_master WHERE type='table' AND name = 'scores';").get();
  if (!table['count(*)']) {
    // If the table isn't there, create it and setup the database correctly.
    sql.prepare("CREATE TABLE scores (id TEXT PRIMARY KEY, user TEXT, guild TEXT, points INTEGER, level INTEGER);").run();
    // Ensure that the "id" row is always unique and indexed.
    sql.prepare("CREATE UNIQUE INDEX idx_scores_id ON scores (id);").run();
    sql.pragma("synchronous = 1");
    sql.pragma("journal_mode = wal");
  }

  // And then we have two prepared statements to get and set the score data.
  client.getScore = sql.prepare("SELECT * FROM scores WHERE user = ? AND guild = ?");
  client.setScore = sql.prepare("INSERT OR REPLACE INTO scores (id, user, guild, points, level) VALUES (@id, @user, @guild, @points, @level);");
});
```

## You get points, and you get points and EVERYBODY GETS POINTS.

Now we can go right ahead and start using the database to retrieve and store points data. We'll be doing this inside the Message handler, and our very first step is to try to retrieve the existing points for a user inside the points table, which would look like this:

```javascript
let score = client.getScore.get(message.author.id, message.guild.id);
```

Importantly, if the bot has never seen this user before they won't have any data, which means we have to "define" their initial values. This can be done with a simple condition, though:

```javascript
if (!score) {
  score = {
    id: `${message.guild.id}-${message.author.id}`,
    user: message.author.id,
    guild: message.guild.id,
    points: 0,
    level: 1
  }
}
```

## Keeping Score

Now that we have our initial "scores" value, we can do two things: first, increment the points. And second, calculate the level of the user.

```javascript
// Increment the score
score.points++;

// Calculate the current level through MATH OMG HALP.
const curLevel = Math.floor(0.1 * Math.sqrt(score.points));

// Check if the user has leveled up, and let them know if they have:
if (score.level < curLevel) {
  // Level up!
  score.level++;
  message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
}
```

And finally, we need to save all this back to the database. SQLite has a great "secret" feature called "INSERT OR REPLACE" and we've already created a prepared statement for this, called `client.setScore`. This will basically _update an existing row with the same_ `id`_, or create a new row if the_ `id` _isn't found_. This explains why we have the `id` field there, in case you were wondering.

```javascript
// This looks super simple because it's calling upon the prepared statement!
client.setScore.run(score);
```

Let's put it all together. Your code should now look like this.

```javascript
const { Client, MessageEmbed } = require("discord.js");
const client = new Client();
const config = require("./config.json");
const SQLite = require("better-sqlite3");
const sql = new SQLite("./scores.sqlite");

client.on("ready", () => {
  // Check if the table "points" exists.
  const table = sql.prepare("SELECT count(*) FROM sqlite_master WHERE type='table' AND name = 'scores';").get();
  if (!table['count(*)']) {
    // If the table isn't there, create it and setup the database correctly.
    sql.prepare("CREATE TABLE scores (id TEXT PRIMARY KEY, user TEXT, guild TEXT, points INTEGER, level INTEGER);").run();
    // Ensure that the "id" row is always unique and indexed.
    sql.prepare("CREATE UNIQUE INDEX idx_scores_id ON scores (id);").run();
    sql.pragma("synchronous = 1");
    sql.pragma("journal_mode = wal");
  }

  // And then we have two prepared statements to get and set the score data.
  client.getScore = sql.prepare("SELECT * FROM scores WHERE user = ? AND guild = ?");
  client.setScore = sql.prepare("INSERT OR REPLACE INTO scores (id, user, guild, points, level) VALUES (@id, @user, @guild, @points, @level);");
});

client.on("messageCreate", message => {
  if (message.author.bot) return;
  let score;
  if (message.guild) {
    score = client.getScore.get(message.author.id, message.guild.id);
    if (!score) {
      score = { id: `${message.guild.id}-${message.author.id}`, user: message.author.id, guild: message.guild.id, points: 0, level: 1 }
    }
    score.points++;
    const curLevel = Math.floor(0.1 * Math.sqrt(score.points));
    if (score.level < curLevel) {
      score.level++;
      message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
    }
    client.setScore.run(score);
  }
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Command-specific code here!
});

client.login(config.token);
```

## Let a user view their level & points

Now we've got the core of this code done, we need to add a few commands, which we can add below all our code in the message handler. We've already separated the arguments and commands, so this will be pretty easy, especially since we've already loaded the `score`, calculated the points, and the level!

```javascript
if (command === "points") {
  return message.reply(`You currently have ${score.points} points and are level ${score.level}!`);
}
```

### Addendum: Leader board and Give commands!

Here are some quick & easy commands you can use, assuming the above code is used and this is still happening in the same file:

```javascript
// You can modify the code below to remove points from the mentioned user as well!
if (command === "give") {
  // Limited to guild owner - adjust to your own preference!
  if (!message.author.id === message.guild.ownerId) return message.reply("You're not the boss of me, you can't do that!");

  const user = message.mentions.users.first() || client.users.cache.get(args[0]);
  if (!user) return message.reply("You must mention someone or give their ID!");

  const pointsToAdd = parseInt(args[1], 10);
  if (!pointsToAdd) return message.reply("You didn't tell me how many points to give...");

  // Get their current points.
  let userScore = client.getScore.get(user.id, message.guild.id);

  // It's possible to give points to a user we haven't seen, so we need to initiate defaults here too!
  if (!userScore) {
    userScore = { id: `${message.guild.id}-${user.id}`, user: user.id, guild: message.guild.id, points: 0, level: 1 }
  }
  userScore.points += pointsToAdd;

  // We also want to update their level (but we won't notify them if it changes)
  let userLevel = Math.floor(0.1 * Math.sqrt(score.points));
  userScore.level = userLevel;

  // And we save it!
  client.setScore.run(userScore);

  return message.channel.send(`${user.tag} has received ${pointsToAdd} points and now stands at ${userScore.points} points.`);
}

if (command === "leaderboard") {
  const top10 = sql.prepare("SELECT * FROM scores WHERE guild = ? ORDER BY points DESC LIMIT 10;").all(message.guild.id);

    // Now shake it and show it! (as a nice embed, too!)
  const embed = new MessageEmbed()
    .setTitle("Leader board")
    .setAuthor(client.user.username, client.user.avatarURL())
    .setDescription("Our top 10 points leaders!")
    .setColor(0x00AE86);

  for (const data of top10) {
    embed.addFields({ name: client.users.cache.get(data.user).tag, value: `${data.points} points (level ${data.level})` });
  }
  return message.channel.send({ embed: [embed] });
}
```
