# Storing Data in a JSON file

In this example we're going to read and write data to and from a JSON file. We'll keep is simple by using this JSON file for a points system. Yes, that's like Mee6 - admitedly that's a piece of shit bot, but people seem to love it so here we are.

> **NOTE**: It should be noted that JSON is not the best storage system for this. It's prone to corruption if you do a lot of read/writes. We'll have an SQLite version coming up soon!

The basis of this system is the `fs` system, for reading and writing the file. And we'll also need the native JSON.stringify\(\) and JSON.parse\(\) functions to convert between the Object and JSON version of our data structure. All these words spining your head around? Get a breath of fresh air and try again!

## Basic Data Structure

So here's an example of an object that contains a list of users, along with their points and level.

```json
{
  "139412744439988224" : { "points": 42, "level": 0 },
  "145978637517193216" : { "points": 3, "level": 0 },
  "90997305578106880" : { "points": 122, "level": 1},
  "173547401905176585" : { "points": 999, "level": 3}
}
```

Simple enough, right? Nothing to it. It's a [JavaScript Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)! But, it's just an example. don't write this just yet.

Instead, create a file in your bot folder called `points.json` with as only content 2 characters: `{}` . Save it and you now have an _empty_ JSON file we can read and write.

## Reading the file

Reading a JSON file is simply a question of loading the file with `fs` module:

```js
const fs = require('fs');

let points = JSON.parse(fs.readFileSync('./points.json', 'utf8'));
```

So at this moment, we have an object called `points` from which we can read any property. So if we pretend for a second that we loaded the above example, we could access `points['139412744439988224'].points` and that would return the number `42`. Great!

> You only need to read the file _once_, when originally loading your bot file, and then as you update it you're just writing to it. This is because the `points` object continues to be updated anyway, we're only using JSON for persistence between reboots!

## Writing to the file

So every time an action happens, we simply increments the proper element in the `points` array and save it. Now let's pretend we go the unoriginal route, and every time someone posts a message, we give them a point!

```js
const fs = require('fs');
let points = JSON.parse(fs.readFileSync('./points.json', 'utf8'));

client.on('message', message => {
  if (message.author.bot) return; // always ignore bots!

  // if the points don't exist, init to 0;
  if (!points[message.author.id]) points[message.author.id] = {
    points: 0,
    level: 0
  };
  points[message.author.id].points++;

  // And then, we save the edited file.
  fs.writeFile('./points.json', JSON.stringify(points), (err) => {
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
  if(message.content.startsWith(prefix + 'level') {
    message.reply(`You are currently level ${curLevel}, with ${userPoints} points.`);
  }
```

## Putting it all together

Ok so we've got a bunch of little bits of code, and your head is probably spinning wonder in what order it goes, right? Well let's fix that now. On top of which we'll simplify a few things. Follow along, now!

```js
const Discord = require('discord.js');
const fs = require('fs');
const client = new Discord.Client();
client.login('your token');

let points = JSON.parse(fs.readFileSync('./points.json', 'utf8'));
const prefix = '+';

client.on('message', message => {
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
    message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
  }

  if (message.content.startsWith(prefix + 'level')) {
    message.reply(`You are currently level ${userData.level}, with ${userData.points} points.`);
  }
  fs.writeFile('./points.json', JSON.stringify(points), (err) => {
    if (err) console.error(err)
  });
});
```
Now take this, and make it **better than Mee6**! Go ahead, I challenge you ;\)
