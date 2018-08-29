---
description: >-
  Here we make a fairly basic points/currency system with automatic leveling,
  manual points giving, and a leaderboard.
---

# SQLite-Based Points System

As mentioned in the [Storing Data in a JSON file](json-based-points-system.md) guide, JSON files could get corrupted due to [_race conditions_](https://en.wikipedia.org/wiki/Race_condition#Software). However SQLite doesn't suffer from that and is a better method of storing data between boot ups than JSON.

That is the focus of this guide: we'll be recreating the points system with SQLite instead of JSON. The core of this system is using the `better-sqlite3` package that you can get from [npmjs.com](https://npmjs.com/package/better-sqlite3).

{% hint style="info" %}
This guide was updated on 2018/03/16 to use `better-sqlite3` which, believe it or not, is a _syncronous_ module for sqlite that's faster than both `sqlite` and `sqlite3`.
{% endhint %}

## Installation

{% hint style="warning" %}
**Pre-Requisites**: `better-sqlite3`, similarly to a lot of modules, gets compiled using `node-gyp` which has 2 very important requirements: Python 2.7 and the C++ Build Tools. For windows, open up an Elevated \(Administrator\) command prompt and run the following FIRST, before installing better-sqlite3: `npm i -g --production windows-build-tools`. For linux, you need `sudo apt-get install build-essential` and you need to figure out how to install Python 2.7 \(NOT Python 3!\) on your system.
{% endhint %}

For this guide to work, you first need to make sure you have the proper modules installed. Let's assume you already have `discord.js` installed, and go straight to installing the sqlite one and its node-gyp dependency:

```text
npm i node-gyp better-sqlite3
```

## Setting the table

Like in the JSON guide you had a data structure, SQLite is no different, but it's called a table. Now what do we want in our table? I'll tell ya!

For this example points system we want the user's ID, points and level. I'm not going to go to deep in to the jargon surrounding SQL and SQLite, but the tables are made from rows and columns of data. Got it? Good, moving on!

Our starting point is a very basic message handler with pre-existing commands - such as what we see in the [Command with Arguments](../first-bot/command-with-arguments.md) page of this guide. The code is as such:

```javascript
const Discord = require("discord.js");
const client = new Discord.Client();
const config = require("./config.json"); // Contains the prefix, and token!

client.on("ready", () => {
  console.log("I am ready!");
});

client.on("message", message => {
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
const sql = new SQLite('./scores.sqlite');
```

We do have a small caveat - we really don't want to react on Direct Messages, so our whole code will be in a block that checks for that. We don't just want to ignore DMs because our bot itself might have DM commands!

```javascript
client.on("message", message => {
  if (message.author.bot) return;
  if (message.guild) {
    // This is where we'll put our code.
  }
  // Rest of message handler
});
```

Your code should look like this now:

```javascript
const Discord = require("discord.js");
const client = new Discord.Client();
const config = require("./config.json");
const SQLite = require("better-sqlite3");
const sql = new SQLite('./scores.sqlite');

client.on("ready", () => {
  console.log("I am ready!");
});

client.on("message", message => {
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

## You get points, and You get points and EVERYBODY GETS POINTS.

Now we can go right ahead and start using the database to retrieve and store points data. We'll be doing this inside the Message handler, and our very first step is to try to retrieve the existing points for a user inside the points table, which would look like this:

```javascript
let score = client.getScore.get(message.author.id, message.guild.id);
```

Importantly, if we've never seen this user before, they will not be seen, which means we have to "defined" their initial values. This can be done with a simple condition, though:

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

Now that we have our initial "Scores" value, we can do two things: first, increment the points. And second, calculate the level of the user.

```javascript
// Increment the score
score.points++;

// Calculate the current level through MATH OMG HALP.
const curLevel = Math.floor(0.1 * Math.sqrt(score.points));

// Check if the user has leveled up, and let them know if they have:
if(score.level < curLevel) {
  // Level up!
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
const Discord = require("discord.js");
const client = new Discord.Client();
const config = require("./config.json");
const SQLite = require("better-sqlite3");
const sql = new SQLite('./scores.sqlite');

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

client.on("message", message => {
  if (message.author.bot) return;
  let score;
  if (message.guild) {
    score = client.getScore.get(message.author.id, message.guild.id);
    if (!score) {
      score = { id: `${message.guild.id}-${message.author.id}`, user: message.author.id, guild: message.guild.id, points: 0, level: 1 }
    }
    score.points++;
    const curLevel = Math.floor(0.1 * Math.sqrt(score.points));
    if(score.level < curLevel) {
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
if(command === "points") {
  return message.reply(`You currently have ${score.points} points and are level ${score.level}!`);
}
```

### Addendum: Leaderboard and Give commands!

Here are some quick & easy commands you can use, assuming the above code is used and this is still happening in the same file:

```javascript
if(command === "give") {
  // Limited to guild owner - adjust to your own preference!
  if(!message.author.id === message.guild.owner) return message.reply("You're not the boss of me, you can't do that!");

  const user = message.mentions.users.first() || client.users.get(args[0]);
  if(!user) return message.reply("You must mention someone or give their ID!");

  const pointsToAdd = parseInt(args[1], 10);
  if(!pointsToAdd) return message.reply("You didn't tell me how many points to give...")

  // Get their current points.
  let userscore = client.getScore.get(user.id, message.guild.id);
  // It's possible to give points to a user we haven't seen, so we need to initiate defaults here too!
  if (!userscore) {
    userscore = { id: `${message.guild.id}-${user.id}`, user: user.id, guild: message.guild.id, points: 0, level: 1 }
  }
  userscore.points += pointsToAdd;

  // We also want to update their level (but we won't notify them if it changes)
  let userLevel = Math.floor(0.1 * Math.sqrt(score.points));
  userscore.level = userLevel;

  // And we save it!
  client.setScore.run(userscore);

  return message.channel.send(`${user.tag} has received ${pointsToAdd} points and now stands at ${userscore.points} points.`);
}

if(command === "leaderboard") {
  const top10 = sql.prepare("SELECT * FROM scores WHERE guild = ? ORDER BY points DESC LIMIT 10;").all(message.guild.id);

    // Now shake it and show it! (as a nice embed, too!)
  const embed = new Discord.RichEmbed()
    .setTitle("Leaderboard")
    .setAuthor(client.user.username, client.user.avatarURL)
    .setDescription("Our top 10 points leaders!")
    .setColor(0x00AE86);

  for(const data of top10) {
    embed.addField(client.users.get(data.user).tag, `${data.points} points (level ${data.level})`);
  }
  return message.channel.send({embed});
}
```

Want a full code example, from top to bottom, so you can lazily copy/paste it? Alright. Sure. Shovel this:

