# Storing Data in an SQLite file

As mentioned in the [Storing Data in a JSON file](/storing-data-in-a-json-file.md) guide, JSON files could get corrupted due to [_race conditions_](https://en.wikipedia.org/wiki/Race_condition#Software). However SQLite doesn't suffer from that and is a better method of storing data between boot ups than JSON.

That is the focus of this guide: we'll be recreating the points system with SQLite instead of JSON. The core of this system is using the `sqlite` \(not `sqlite3`\) package that you can get from [npmjs.com](http://www.npmjs.com/).

> **NOTE:** The reason I stated `sqlite` and not `sqlite3` is because `sqlite` is a promise wrapper for `sqlite3` and with Discord.js being promise based, it's obviously the best choice.

## Setting the table

Like in the JSON guide you had a data structure, SQLite is no different, but it's called a table. Now what do we want in our table? I'll tell ya!

For this example points system we want the user's ID, points and level, I'm not going to go to deep in to the jargon surrounding SQL and SQLite, but the tables are made from rows and columns of data. Got it? Good, moving on!

Let's take the core elements from the the example bot on [getting started](/getting-started/the-long-version.md).

```js
const Discord = require('discord.js');
const client = new Discord.Client();

client.login('yourawesomesecrettoken');

client.on('ready', () => {
    console.log('Ready!');
});

client.on('message', message => {
  if (message.author.bot) return; // Ignore bots.
});
```

Now we've got that we should `require` sqlite and make use of it, put the following under `const client`

```js
const sql = require('sqlite');
sql.open('./score.sqlite');
```

> **NOTE:** Don't worry about the file, we'll be doing a special conditional shortly that'll do some fancy magic!

Alright now that's required correctly we want to prevent people trying to DM the bot to increase their points. Add the following code below the ignore bots line.

```js
if (message.channel.type === 'dm') return; // Ignore DM channels.
```

Your code should look like this now;

```js
const Discord = require('discord.js');
const client = new Discord.Client();
const sql = require('sqlite');
sql.open('./score.sqlite');

client.login('yourawesomesecrettoken');

client.on('ready', () => {
  console.log('Ready!');
});

client.on('message', message => {
  if (message.author.bot) return; // Ignore bots.
  if (message.channel.type === 'dm') return; // Ignore DM channels.
});
```

## Alright, time to get down to business.

We need to start the sqlite chain, we don't have to worry about opening the database, as it's opened at the top of our file, so it's loaded when we need it. With sqlite being promise based, we need to start off with a `get` query `then` follow it up with a `catch`

```js
sql.get(`SELECT * FROM scores WHERE userId ='${message.author.id}'`).then(row => {

}).catch(() => {

});
```

Right, now we've got that out of the way, we need to add our logic to it, but let's work on the catch first so, remember when I mentioned about some magic? Well, when the code attempted to open the database it in fact created the file for us since there was no file. However the file doesn't have a table, that's where the catch comes in, add the following code to the catch.

```js
console.error; // Gotta log those errors
sql.run('CREATE TABLE IF NOT EXISTS scores (userId TEXT, points INTEGER, level INTEGER)').then(() => {
  sql.run('INSERT INTO scores (userId, points, level) VALUES (?, ?, ?)', [message.author.id, 1, 0]);
});
```

Let's break that code down shall we?

Obviously the first thing we want to do on errors is to log them, but we also want to do some magic... what the code does is creates the _scores_ table in the database file IF it does not exist, then it inserts the same data you would get if it found the file, otherwise it just opens and inserts the data just like normal.

On to the next bit of logic, inside the `then` you should notice we defined `row`, now we'll put it into good use with the following code.

```js
if (!row) { // Can't find the row.
  sql.run('INSERT INTO scores (userId, points, level) VALUES (?, ?, ?)', [message.author.id, 1, 0]);
} else { // Can find the row.
  sql.run(`UPDATE scores SET points = ${row.points + 1} WHERE userId = ${message.author.id}`);
}
```

Now, that will either insert \(if no row is found\), or update the authors points if it found a row... perfect!

Your code should now look like this.

```js
const Discord = require('discord.js');
const client = new Discord.Client();
const sql = require('sqlite');
sql.open('./score.sqlite');

client.login('yourawesomesecrettoken');

client.on('message', message => {
  if (message.author.bot) return;
  if (message.channel.type !== 'text') return;
  sql.get(`SELECT * FROM scores WHERE userId ='${message.author.id}'`).then(row => {
    if (!row) {
      sql.run('INSERT INTO scores (userId, points, level) VALUES (?, ?, ?)', [message.author.id, 1, 0]);
    } else {
      sql.run(`UPDATE scores SET points = ${row.points + 1} WHERE userId = ${message.author.id}`);
    }
  }).catch(() => {
    console.error;
    sql.run('CREATE TABLE IF NOT EXISTS scores (userId TEXT, points INTEGER, level INTEGER)').then(() => {
      sql.run('INSERT INTO scores (userId, points, level) VALUES (?, ?, ?)', [message.author.id, 1, 0]);
    });
  });
});
```

## DING Level up!

Now, what's the point of having all of these points? To level up of course!

But first we need to calculate the level, so I'm going to take the code from the original article and rework it slightly.

This is the original code.

```js
  let userLevel = points[<Message>.author.id] ? points[<Message>.author.id].level : 0;
  if(userLevel < curLevel) {
    // Level up!
    <Message>.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
  }
```

Now, we need to change a couple of things, such as using sql row names, but through the magic of tutorials, here's what the code should look like after our tweaks

```js
let curLevel = Math.floor(0.1 * Math.sqrt(row.points + 1));
if (curLevel > row.level) {
  row.level = curLevel;
  sql.run(`UPDATE scores SET points = ${row.points + 1}, level = ${row.level} WHERE userId = ${message.author.id}`);
  message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
}
```

Place that code inside our `row` conditional, above the `update` query.

## Let a user view their level & points.

Now we've got the core of this code done, we need to add a few commands, so as normal we'll add a prefix and ignore messages without a prefix.

Place this above your message handler

```js
const prefix = '+';
```

And this just below the `sql` code block

```js
if (!message.content.startsWith(prefix)) return; // Ignore messages that don't start with the prefix

if (message.content.startsWith(prefix + 'level') {

} else

if (message.content.startsWith(prefix + 'points') {

}
```

Now for the commands. We want to view the row `level` and `points`, so we need to do another sql query like we did above, then respond to the user with their level or points.

All you'll need to do is swap `level` for `points` and the response message and you're set!

```js
sql.get(`SELECT * FROM scores WHERE userId ='${message.author.id}'`).then(row => {
  if (!row) return message.reply('Your current level is 0');
  message.reply(`Your current level is ${row.level}`);
});
```

## Conclusion

After this guide, your code should look like this;

```js
const Discord = require('discord.js');
const client = new Discord.Client();
const sql = require('sqlite');
sql.open('./score.sqlite');

client.login('yourawesomesecrettoken');

const prefix = '+';
client.on('message', message => {
  if (message.author.bot) return;
  if (message.channel.type !== 'text') return;

  sql.get(`SELECT * FROM scores WHERE userId ='${message.author.id}'`).then(row => {
    if (!row) {
      sql.run('INSERT INTO scores (userId, points, level) VALUES (?, ?, ?)', [message.author.id, 1, 0]);
    } else {
      let curLevel = Math.floor(0.1 * Math.sqrt(row.points + 1));
      if (curLevel > row.level) {
        row.level = curLevel;
        sql.run(`UPDATE scores SET points = ${row.points + 1}, level = ${row.level} WHERE userId = ${message.author.id}`);
        message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
      }
      sql.run(`UPDATE scores SET points = ${row.points + 1} WHERE userId = ${message.author.id}`);
    }
  }).catch(() => {
    console.error;
    sql.run('CREATE TABLE IF NOT EXISTS scores (userId TEXT, points INTEGER, level INTEGER)').then(() => {
      sql.run('INSERT INTO scores (userId, points, level) VALUES (?, ?, ?)', [message.author.id, 1, 0]);
    });
  });

  if (!message.content.startsWith(prefix)) return;

  if (message.content.startsWith(prefix + 'level')) {
    sql.get(`SELECT * FROM scores WHERE userId ='${message.author.id}'`).then(row => {
      if (!row) return message.reply('Your current level is 0');
      message.reply(`Your current level is ${row.level}`);
    });
  } else

  if (message.content.startsWith(prefix + 'points')) {
    sql.get(`SELECT * FROM scores WHERE userId ='${message.author.id}'`).then(row => {
      if (!row) return message.reply('sadly you do not have any points yet!');
      message.reply(`you currently have ${row.points} points, good going!`);
    });
  }
});
```

Now, when ever anyone in your guild talks, the code will either create a new table row for them, or update their table role by taking the current amount of points and simply adding 1 to it. My challenge for you dear reader, is to make this multi-guild friendly.

