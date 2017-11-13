# Enmap-Based Points System

Like in the previous _"Points System"_ articles, this one will follow the same path with cleaner, simpler module! _ʸᵃᵃᵃᵃᵃʸ_.

But we're using [Enmap](/coding-guides/using-persistentcollections.md) by Evie.Codes, which comes natively with [GuideBot](https://github.com/An-Idiots-Guide/guidebot) and it's what we will be using in this article.

> This guide has been updated to the 0.4.0+ version of Enmap, which introduces providers. Make sure you're using the right version! Run `npm ls enmap` to check your version.

To start with, we need to open up index.js and add two different things. First, we need to import and initialize the *Provider* itself. Then, you need to create a new persistent Enmap using the provider itself. Here's how it goes:

```js
const Enmap = require("enmap");
const EnmapLevel = require("enmap-level");

const pointProvider = new EnmapLevel({name: "points"});
this.points = new Enmap({provider: pointProvider});
```

That will create a new Enmap under the name of points, and attaches it to the client extention so it can be used where ever you have access to the client object.

Now before we move to the `/modules/functions.js` file to add a _"monitor"_, we want to add this line to the message event, just below `if (message.author.bot) return`:

```js
client.pointsMonitor(client, message);
```

The line above will pass the client and message objects to our pointsMonitor function we're about to create, both are needed so we can get the points database, and the message.

>***NOTE:*** The reason we're making a _"monitor"_ is because if we add this code straight to the message event, it would render the bot useless, you'll see why in a moment.

## Points Monitor

Now let's move onto the `/modules/functions.js` file and start a new function with the following.

```js
client.pointsMonitor = (client, message) => {

};
```

You should make sure that no one can DM spam the bot, so throw this on the next line under the opening line.

```js
if (message.channel.type !=='text') return;
```

We want to be able to grab the bot settings based on the guilds ID, so add the following inside the function.

```js
const settings = client.settings.get(message.guild.id);
```

Remember earlier when I mentioned this code would render the bot useless if we put it in the message event, the following code is why.

```js
if (message.content.startsWith(settings.prefix)) return;
```

Naturally, if we used that with the bot ignoring messages that **DIDN'T** start with the prefix, the bot would ignore **every** message, which would have rendered it useless.

So far so good, let's move on to the actual points system, under the bottom if statement, you should go ahead and add the following.

```js
  const score = client.points.get(message.author.id) || { points: 0, level: 0 };
  score.points++
```

Alright, the top line basically grabs your previous points from the PCollection, OR if you haven't gained any points yet, will give you the blank template.

The second line will take the `points` property of `score` and increases it by 1 on every message it sees.

Now we have to do some math, because what's the point of collecting all of these points if you have nothing to show off, so the math is to calculate what **level** you should be at.

```js
const curLevel = Math.floor(0.1 * Math.sqrt(score.points));
```

This line will calculate the square root of `score.points` then multiplies that result by 0.1 then floors that result for a round number.

Now we should work out if you've amassed enough points to actually level up, so we have an if statement.

```js
if (score.level < curLevel) {

}
```

That basically compares your saved level (`score.level`), with your _"current"_ level, and we want to give the users a nice message if they have leveled up, so throw in a nice reply message.

```js
message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
```

Lastly, we want to update the `score.level` value with the new level so throw this under the `message.reply`.

```js
score.level = curLevel;
```

Now finally you should save the new score object to the user so just above the function closing brace, add this line of code.

```js
client.points.set(message.author.id, score);
```

This is what your code should look like. If it doesn't, that's fine, everyone makes mistakes.

```js
client.pointsMonitor = (client, message) => {
  if (message.channel.type !=='text') return;
  const settings = client.settings.get(message.guild.id);
  if (message.content.startsWith(settings.prefix)) return;
  const score = client.points.get(message.author.id) || { points: 0, level: 0 };
  score.points++;
  const curLevel = Math.floor(0.1 * Math.sqrt(score.points));
  if (score.level < curLevel) {
    message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
    score.level = curLevel;
  }
  client.points.set(message.author.id, score);
};
```

## Points & Level Commands

Alright, that's the bulk of the code, you could throw this into your GuideBot based bot and it would work like a charm, however your users wouldn't know how many points, or even their levels, so let's fix that, make two new commands, `/commands/points.js` and `/commands/level.js` respectfully.

Let's start with the points command, as that is really simple to throw together, grab this blank template for the command.

```js
exports.run = async (client, message) => {

};

exports.conf = {
  hidden: false,
  guildOnly: true,
  aliases: [],
  permLevel: 0
};

exports.help = {
  name: '',
  description: '',
  usage: '',
  category: '',
  extended: ''
};
```

Then just fill in the conf and help exports, I would suggest leaving `guildOnly` set to `true` otherwise that may cause issues, i.e. people DM spamming the bot to inflate their points.

Inside the exports.run, you'll need to grab the points of the message author, to do that do the following;

```js
const scorePoints = client.points.get(message.author.id).points;
```

That will grab the users points, now we want to make sure to give an output regardless if the user has points or not, add the following below the line you just wrote.

```js
!scorePoints ? message.channel.send('You have no points yet.') : message.channel.send(`You have ${scorePoints} points!`);
```

The inside of your `exports.run` should look like this now.

```js
exports.run = async (client, message) => {
  const scorePoints = client.points.get(message.author.id).points;
  !scorePoints ? message.channel.send('You have no points yet.') : message.channel.send(`You have ${scorePoints} points!`);
};
```

Now, let's copy the points command and paste it inside the level command and change `points` to `level`, this is how your level's `exports.run` should look.

```js
exports.run = async (client, message) => {
  const scoreLevel = client.points.get(message.author.id).level;
  !scoreLevel ? message.channel.send('You have no levels yet.') : message.channel.send(`You are currently level ${scoreLevel}!`);
};
```

And there you have it, you've successfully created a points system using Enhanced Maps. Now there's a few caveats to this system... the current code is limited to users, not guilds so if a user wanted to boost their score, they can invite the bot to a private guild and spam the ever loving snot out of it to boost their score. Note also that Enmap currently does not support sharding or calling the points database from multiple files (though a fix is planned for this limitation!)

In the famous words of Evie _"Now take this, and make it **better than Mee6!** Go ahead, I challenge you ;)"_ - Evie.Codes