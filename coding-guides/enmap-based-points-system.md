# Enmap-Based Points System

Similarly to the other _"Points System"_ articles, let's explore another altenative with a cleaner, simpler module! _ʸᵃᵃᵃᵃᵃʸ_.

[Enmap](https://npmjs.org/package/enmap) was created by Evie.Codes specifically to simplify some of the code involved in the use and maintenance of a database with a "cached version" in memory. In other words, Enmap is both a Collection-inspired structure, as well as a database wrapper that saves stuff automatically. You don't even have to worry about it! Enmap is used in [GuideBot](https://github.com/AnIdiotsGuide/guidebot), and has been downloaded well over 30,000 times from NPM.

## Installing and Importing

### Pre-Requisites

{% hint style="warning" %}
`enmap-sqlite` has important pre-requisites before installing. Please see the instructions below for your operating system.
{% endhint %}

{% tabs %}
{% tab title="Windows" %}
On Windows, two things are required to install enmap-sqlite. Python 2.7 and the Visual Studio C++ Build Tools. They are required for any module that is _built_ on the system, which includes sqlite. 

> The Windows Built Tools require over 3GB of space to install and use. Make sure you have enough space before starting this download and install!

To install the necessary pre-requisites on Windows, the easiest is to simply run the following command, _under an **administrative** command prompt or powershell:_

```javascript
npm i -g --production windows-build-tools
```

> It's _very important_ that this be run in the **administrative** prompt, and not a regular one.

Once the windows-build-tools are installed \(this might take quite some time, depending on your internet connection\), close all open command prompts, powershell windows, and editors with a built-in console/prompt. Otherwise, the next command will not work. 
{% endtab %}

{% tab title="Linux" %}


On Linux, the pre-requisites are much simpler in a way. A lot of modern systems \(such as Ubuntu, since 16.04\) already come with python 2.7 pre-installed. For some other systems, you might have to fiddle with it to either get python 2.7 installed, or to install both 2.7 and 3.x simultaneously. Google will be your friend. 

As for the C++ build tools, that's installed using the simple command: `sudo apt-get install build-essential` for most debian-based systems. For others, look towards your package manager and specificall "GCC build tools". Your mileage may vary but hey, you're using Linux, you should know this stuff. 
{% endtab %}

{% tab title="Mac OS" %}
As of writing this page, MacOS versions seem to all come pre-built with Python 2.7 on the system. You will, however, need the C++ build tools. 

* Install [XCode](https://developer.apple.com/xcode/download/)
* Once XCode is installed, go to **Preferences**, **Downloads**, and install the **Command Line Tools**.

Once installed, you're ready to continue. 
{% endtab %}
{% endtabs %}

### Installing

Let's start with installing the 2 parts that we need for this to work: `enmap` and `enmap-sqlite`. Simply run the following command in your project folder:

```text
npm i enmap enmap-sqlite
```

Once that's complete, we need to open up index.js and add two different things. First, we need to import and initialize the _Provider_ itself. Then, you need to create a new persistent Enmap using the provider itself. Here's how it goes:

```javascript
const Enmap = require("enmap");
const Provider = require("enmap-sqlite");

client.points = new Enmap({provider: new Provider({name: "points"})});
```

That will create a new Enmap under the name of points, and attaches it to the client extention so it can be used where ever you have access to the client object.

## Accumulating Points

The obvious goal of a points system is to accumulate fake internet points and gloat about it. So, of course, that's going to be our first focus. In this example implementation, we will make the points guild-specific, and unuseable in DMs. Points will still accumulate even if the user does a command, which simplifies our code a bit.

Our starting point is a very basic message handler with pre-existing commands - such as what we see in the [Command with Arguments](../first-bot/command-with-arguments.md) page of this guide. The code is as such:

```javascript
client.on("message", message => {
  if (message.author.bot) return;
  // This is where we'll put our code.
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Command-specific code here!
});
```

We do have a small caveat - we really don't want to react on Direct Messages, so our whole code will be in a block that checks for that:

```javascript
client.on("message", message => {
  if (message.author.bot) return;
  if (message.guild) {
    // This is where we'll put our code.
  }
  // Rest of message handler
});
```

Our very first step is going to be to initialize a new entry in the enmap for any new user - one we haven't received a message from before. This is done using the `enmap.has(key)` method, which can check if a specific key exists in the Enmap, and `enmap.set(key, value)` method that adds data to Enmap. Note that our keys take the format of `guildid-userid` so they're unique to the guild and the user. Also, our data in this case is a complete object, which we'll take advantage of with Enmap 2.0.

```javascript
client.on("message", message => {
  if (message.author.bot) return;
  if (message.guild) {
    if(!client.points.has(`${message.guild.id}-${message.author.id}`)) {
      client.points.set(`${message.guild.id}-${message.author.id}`, {
        user: message.author.id,
        guild: message.guild.id,
        points: 0,
        level: 1
      });
    }
    // Code continued...
  }
  // Rest of message handler
});
```

There's obviously a few ways we could have done this, including some fancy ternary condition or whatever. I will, however, keep this code as simple to read as possible.

Two methods will be used for the following bit, `enmap.getProp(key, propname)` which retrieves, specifically, only the points property from the saved object. We'll also use `enmap.setProp(key, propname, value)` which will then _save_ that points property back into the enmap without needing to load the whole object.

```javascript
client.on("message", message => {
  if (message.author.bot) return;
  if (message.guild) {
    // Let's simplify the `key` part of this.
    const key = `${message.guild.id}-${message.author.id}`;
    if(!client.points.has(key)) {
      client.points.set(key, {
        user: message.author.id, guild: message.guild.id, points: 0, level: 1
      });
    }
    let currentPoints = client.points.getProp(key, "points");
    client.points.setProp(key, "points", ++currentPoints);
  }
  // Rest of message handler
});
```

## Ding!

Time to level up! If a user has enough points, they will go up a level. Now we have to do some math here, but don't run off in fear, this one's pretty easy. This is how we calculate the levels:

```javascript
const curLevel = Math.floor(0.1 * Math.sqrt(currentPoints));
```

This line will calculate the square root of `currentPoints` then multiplies that result by 0.1 then floors that result for a round number.

Now we should work out if you've amassed enough points to actually level up, by grabbing the current user's level and comparing them. If the new calculated level is higher, it means the user leveled up and we can deal with that, first by sending them a very annoying mee6-inspired message!

```javascript
if (client.points.getProp(key, "level") < curLevel) {
  message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
}
```

Lastly, we want to update the `score.level` value with the new level so throw this under the `message.reply`.

```javascript
client.points.setProp (key, "level", curLevel);
```

So here's the whole thing from top to bottom, with bonus comments!

```javascript
client.on("message", message => {
  // As usual, ignore all bots.
  if (message.author.bot) return;

  // If this is not in a DM, execute the points code.
  if (message.guild) {
    // We'll use the key often enough that simplifying it is worth the trouble.
    const key = `${message.guild.id}-${message.author.id}`;

    // Triggers on new users we haven't seen before.
    if(!client.points.has(key)) {
      // The user and guild properties will help us in filters and leaderboards.
      client.points.set(key, {
        user: message.author.id, guild: message.guild.id, points: 0, level: 1
      });
    }

    // Get only the current points for the user.
    let currentPoints = client.points.getProp(key, "points");

    // Increment the points and save them.
    client.points.setProp(key, "points", ++currentPoints);

    // Calculate the user's current level
    const curLevel = Math.floor(0.1 * Math.sqrt(currentPoints));

    // Act upon level up by sending a message and updating the user's level in enmap.
    if (client.points.getProp(key, "level") < curLevel) {
      message.reply(`You've leveled up to level **${curLevel}**! Ain't that dandy?`);
      client.points.setProp (key, "level", curLevel);
    }
  }
  // Rest of message handler
});
```

## Points & Level Commands

Alright, that's the bulk of the code, you could throw this into your bot and it would work like a charm, however your users wouldn't know how many points, or even their levels, so let's fix that, make a new command called `points`, which will also show them their level.

{% hint style="info" %}
Obviously there's no way for us to know how you're making commands, so again we'll assume you're doing a bot in a single js file. You may need to adjust the code, of course!
{% endhint %}

So let's re-iterate our current starting position.

```javascript
client.on("message", message => {
  if (message.author.bot) return;
  if (message.guild) { /* Points Code Here */ }
  if (message.content.indexOf(config.prefix) !== 0) return;

  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // Command-specific code here!
});
```

The `points` command would look like this:

```javascript
  if (command === "points") {
    return message.channel.send(`You currently have ${client.points.getProp(key, "points")}, and are level ${client.points.getProp(key, "level")}!`);
  }
```

## We are the champions, my friend!

Let's finish this off with a very simple `leaderboard` command that will show the top 10 users in the current guild. For this we'll need to _filter_ the Enmap to only get the users for the current guild, then we'll convert the results to an array, sort that, and keep the first 10 results only.

{% hint style="info" %}
We convert to an array because an `Enmap`, just like its underlying `Map` structure, is not ordered and thus cannot be sorted. It may _seem_ ordered because it stores by keys, but that's actually a quirk, not a feature.
{% endhint %}

So here's our leaderboard command:

```javascript
if(command === "leaderboard") {
  // Get a filtered list (for this guild only), and convert to an array while we're at it.
  const filtered = client.points.filterArray( p => p.guild === message.guild.id );

  // Sort it to get the top results... well... at the top. Y'know.
  const sorted = filtered.sort((a, b) => a.points < b.points);

  // Slice it, dice it, get the top 10 of it!
  const top10 = sorted.splice(0, 10);

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

In the famous words of Evie _"Now take this, and make it **better than Mee6!** Go ahead, I challenge you ;\)"_ - Evie.Codes

### ADDENDUM: Extra Commands!

Taken straight from [Evie's Gist on points](https://gist.github.com/eslachance/12e2239aa353b350e075a4b006238335) \(but adjusted for Enmap 2.0\), you might find these useful!

```javascript
  if(command === "give") {
    // Limited to guild owner - adjust to your own preference!
    if(!message.author.id === message.guild.owner) return message.reply("You're not the boss of me, you can't do that!");

    const user = message.mentions.users.first() || client.users.get(args[0]);
    if(!user) return message.reply("You must mention someone or give their ID!");

    const pointsToAdd = parseInt(args[1], 10);
    if(!pointsToAdd) return message.reply("You didn't tell me how many points to give...")

    // Get their current points.
    const userPoints = client.points.getProp(key, "points");
    userPoints += pointsToAdd;

    // And we save it!
    client.points.setProp(key, "points", userPoints)

    message.channel.send(`${user.tag} has received ${pointstoAdd} points and now stands at ${userPoints} points.`);
  }

  if(command === "cleanup") {
    // Let's clean up the database of all "old" users, and those who haven't been around for... say a month.
    // This will require you to add the following in the points code above: client.points.setProp(key, "lastSeen", new Date());

    // Get a filtered list (for this guild only).
    const filtered = client.points.filter( p => p.guild === message.guild.id );

    // We then filter it again (ok we could just do this one, but for clarity's sake...)
    // So we get only users that haven't been online for a month, or are no longer in the guild.
    const rightNow = new Date();
    const toRemove = filtered.filter(data => {
      return !message.guild.members.has(data.user) || rightNow - 2592000000 > data.lastSeen;
    });

    toRemove.forEach(data => {
      client.points.delete(`${message.guild.id}-${data.user}`);
    });

    message.channel.send(`I've cleaned up ${toRemove.size} old farts.`);
  }
```

