# Sharding

**Sharding** is the method by which a bot's code is "split" into multiple _instances_ of itself. When a bot is sharded, each shard handles only a certain percentage of all the guilds the bot is on.

{% hint style="info" %}
You do not need to worry about sharding until your bot hits around 2,400 guilds. YOU MUST SHARD before you hit 2,500 guilds, however.
{% endhint %}

## Sharding Caveats

There are additional difficulties when sharding a bot that add complexity to your code \(one of the reasons you shouldn't shard too early\).

* Collections do not cache data from all shards, so you can't grab data from a guild in another shard easily.
* In order to do anything across shards you need to worry about using `broadcastEval` and such \(tutorial coming soon!\).
* Sharded bots often gain very marginal performance increase and might even use _more_ memory due to using more node processes.
* If you're using any sort of database or connection, multiple shards may cause issues with multiple processes connecting to a single end point.

## Example Sharding Manager Code

```javascript
/*
    The following code goes into it's own file, and you run this file
    instead of your main bot file.
*/

// Include discord.js ShardingManger
const { ShardingManager } = require('discord.js')

// Create your ShardingManger  instance
const manager = new ShardingManager('./YOUR_BOT_FILE_NAME.js', {
    // for ShardingManager options see:
    // https://discord.js.org/#/docs/main/stable/class/ShardingManager
    totalShards: 'auto',
    token: 'YOUR_TOKEN_GOES_HERE'
})

// Spawn your shards
manager.spawn()

// Emitted when a shard is created
manager.on('shardCreate', (shard) => console.log(`Shard ${shard.id} launched`))
```

