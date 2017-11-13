# Storing Data in a JSON file

This page has been removed due to a number of people having issues with JSON storage becoming corrupted when using it as a database.

## Why is JSON prone to corruption?

The answer is: it's not *inherently* a problem. JSON is fine for storing data that you access often. JSON is fine as a transfer system between services and apps.

The issue isn't with JSON. The issue is with `fs`. So really, this applies not just to JSON but also to TXT files or any other plain text file you're attempting to write to.

See here's the thing: If you're attempting to simultaneously **read** a file and **write** to it at the same time, or if you're **writing** to a file from **more than one location**, the file risks being corrupted. And the more this happens, the higher the chances. It might work for small bots, but as it grows you are going to lose that data.

## Does this mean JSON is very bad?

Not for all purposes. As we cover in [Adding a Config File](/getting-started/config-json-file.md), you can store *static* data that generally is modified manually and applied on bot reboot. That data can be as big as you want - it can be just 4 keys, it can be 1000 entries in a random text thing... the fact is it's being loaded once and then not written to.

## So what do I do now, how do I store data?

You have **so many** good alternatives to using JSON. And some are covered right here.

### If your bot is not sharded?

If you want a simple `key/value` pair storage, like ID + Object, you can use `enmap`, which is covered in [Introducing Enhanced Maps](/coding-guides/using-persistentcollections.md).

If you need a little more umph, like multiple tables, multiple keys, etc, you can use `sqlite`. The [SQLite-Based Points System](/coding-guides/storing-data-in-an-sqlite-file.md) page shows you how to do a points system but of course, SQLite can be used for much more than this.

### If your bot is sharded or the database is shared

For more stable, multi-process access, think of getting a bigger database system. While those systems will require the installation of a database server, they will offer much more capabilities, power, and reliability.

A few options are [rethinkdbdash](https://www.npmjs.com/package/rethinkdbdash), [mysql](https://www.npmjs.com/package/mysql), [redis](https://www.npmjs.com/package/redis). You could also use an ORM if you're into this sort of thing, [sequelize](https://www.npmjs.com/package/sequelize) is a highly recommended package!

# PAGE RESTORED FOR ARCHIVES.

## The original page contents are listed below.

## Writing to the file

So every time an action happens, we simply increments the proper element in the `points` array and save it. Now let's pretend we go the unoriginal route, and every time someone posts a message, we give them a point!

```js
const fs = require("fs");
let points = JSON.parse(fs.readFileSync("./points.json", "utf8"));

client.on("message", message => {
  if (message.author.bot) return; // always ignore bots!

  // if the points don"t exist, init to 0;
  if (!points[message.author.id]) points[message.author.id] = {
    points: 0,
    level: 0
  };
  points[message.author.id].points++;

  // And then, we save the edited file.
  fs.writeFile("./points.json", JSON.stringify(points), (err) => {
    if (err) console.error(err)
  });
});
```

And now, we can access the points of a user by grabbing from the object. However, if a user doesn't have points we want to show 0 instead, obviously. So, `let userpoints = points[message.author.id] ? points[message.author.id] : 0;` \(if you don't know what the : and ? shenanigans means, look up [Ternary Operator Assignment](http://stackoverflow.com/questions/5080242/javascript-ternary-operator-and-assignment)!

## Calculating Levels

So I put in the `levels` property in there because why else would you have points, right? Here, we have a little bit of math. Don't be scared, it's pretty simple!

```js
  let userPoints = points[message.author.id] ? points[message.author.id].points : 0;
  let curLevel = Math.floor(0.1 * Math.sqrt(userPoints));
```

Alright, so we have a level. Let's do like all the lame bots out there and output a message when a new level is reached! yay.

```js
  let userLevel = points[message.author.id] ? points[message.author.id].level : 0;
  if(userLevel < curLevel) {
    // Level up!
    message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
  }
```

## Letting a user see their level

Ok I'm certainly not going to give you the secret recipe to show a full profile like Tatsumaki. But, I can at least show you how to return a really basic command that loads and shows it.

```js
  if(message.content.startsWith(prefix + "level")) {
    message.reply(`You are currently level ${curLevel}, with ${userPoints} points.`);
  }
```

## Putting it all together

Ok so we've got a bunch of little bits of code, and your head is probably spinning wonder in what order it goes, right? Well let's fix that now. On top of which we'll simplify a few things. Follow along, now!

```js
const Discord = require("discord.js");
const fs = require("fs");
const client = new Discord.Client();

let points = JSON.parse(fs.readFileSync("./points.json", "utf8"));
const prefix = "+";

client.on("message", message => {
  if (!message.content.startsWith(prefix)) return;
  if (message.author.bot) return;

  if (!points[message.author.id]) points[message.author.id] = {
    points: 0,
    level: 0
  };
  let userData = points[message.author.id];
  userData.points++;

  let curLevel = Math.floor(0.1 * Math.sqrt(userData.points));
  if (curLevel > userData.level) {
    // Level up!
    userData.level = curLevel;
    message.reply(`You"ve leveled up to level **${curLevel}**! Ain"t that dandy?`);
  }

  if (message.content.startsWith(prefix + "level")) {
    message.reply(`You are currently level ${userData.level}, with ${userData.points} points.`);
  }
  fs.writeFile("./points.json", JSON.stringify(points), (err) => {
    if (err) console.error(err)
  });

});

client.login("SuperSecretBotTokenHere");
```

Now take this, and make it **better than Mee6**! Go ahead, I challenge you ;\)
