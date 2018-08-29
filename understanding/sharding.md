---
description: >-
  A very basic guide on how to shard your bot. If you don't have at least 2000
  guilds on your bot, you don't need this page.
---

# Sharding

**Sharding** is the method by which a bot's code is "split" into multiple _instances_ of itself. When a bot is sharded, each shard handles only a certain percentage of all the guilds the bot is on.

{% hint style="info" %}
You do not need to worry about sharding until your bot hits around 2,400 guilds. YOU MUST SHARD before you hit 2,500 guilds, however.
{% endhint %}

## Sharding Caveats

There are additional difficulties when sharding a bot that add complexity to your code \(one of the reasons you shouldn't shard too early\).

* Collections do not cache data from all shards, so you can't grab data from a guild in another shard easily.
* In order to do anything across shards you need to worry about using `broadcastEval` and such \(tutorial comming soon!\).
* Sharded bots often gain very marginal performance increase and might even use _more_ memory due to using more node processes.
* If you're using any sort of database or connection, multiple shards may cause issues with multiple processes connecting to a single end point.

## Example Sharding Code

```javascript
/*
    The following code goes into it's own file, and you run this file
    instead of your main bot file.
*/

const Discord = require('discord.js');
const Manager = new Discord.ShardingManager('./YOUR_BOT_FILE_NAME.js');
Manager.spawn(2); // This example will spawn 2 shards (5,000 guilds);
```

