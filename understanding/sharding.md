# Sharding

**Sharding** is the method by which a bot's code is "split" into multiple _instances_ of itself. When a bot is sharded, each shard handles only a certain percentage of all the guilds the bot is on.

{% hint style="info" %}
You do not need to worry about sharding until your bot hits around 2,400 guilds. YOU MUST SHARD before you hit 2,500 guilds, however.
{% endhint %}

## Sharding Caveats

There are additional difficulties when sharding a bot that add complexity to your code \(one of the reasons you shouldn't shard too early\).

* Collections do not cache data from all shards, so you can't grab data from a guild in another shard easily.
* In order to do anything across shards you need to worry about using `broadcastEval` and such \(Examples and explanation below\).
* Sharded bots often gain very marginal performance increase and might even use _more_ memory due to using more node processes.
* If you're using any sort of database or connection, multiple shards may cause issues with multiple processes connecting to a single end point.

## Example Sharding Manager Code

```javascript
/*
    The following code goes into it's own file, and you run this file
    instead of your main bot file.
*/

// Include discord.js ShardingManger
const { ShardingManager } = require('discord.js');

// Create your ShardingManger instance
const manager = new ShardingManager('./YOUR_BOT_FILE_NAME.js', {
    // for ShardingManager options see:
    // https://discord.js.org/#/docs/main/stable/class/ShardingManager
    totalShards: 'auto',
    token: 'YOUR_TOKEN_GOES_HERE'
});

// Spawn your shards
manager.spawn();

// Emitted when a shard is created
manager.on('shardCreate', (shard) => console.log(`Shard ${shard.id} launched`));
```

## Sharing Information Between Shards

{% hint style="info" %}
Information is not readily available between shards. In order to get or share information across shards, you will need to make use of either `fetchClientValues()` or `broadcastEval()`.
{% endhint %}

Responses from [`fetchClientValues`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=fetchClientValues) and [`broadcastEval`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=broadcastEval) return a Promise which resolves to an Array containing the values from each shard. See example data for each below.

## FetchClientValues

[`fetchClientValues`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=fetchClientValues) gets Client properties from all shards. This is used when you would like to get any of the nested properties of the Client, such as `guilds.size` or `uptime`.

Example data from `fetchClientValues()`:
```javascript
/*
	Example of result of fetchClientValues() on a bot with 4,300 guilds split across 4 shards
*/
const res = await client.shard.fetchClientValues('guilds.size');

console.log(res);
// 	Array: [
//		1075,	// shard 0
//		1075,	// shard 1
//		1075,	// shard 2
//		1075	// shard 3
//	]

````


Example `fetchClientValues()` function:
```javascript
/*
  	Example by ZiNc#2032
  	The following code fetches total combined shards' server counts
  
  	discord.js version 11.x
  	client = new discordjs.Client()

  	returns Number
*/

const getServerCount = async () => {
    // get guild collection size from all the shards
    const req = await client.shard.fetchClientValues('guilds.size');

    // return added value
    return req.reduce((p, n) => p + n, 0);
}
```

## BroacastEval

[`broadcastEval`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=broadcastEval) evaluates the input in the context of each shard's Client(s). This is used when you want to execute a method or process data on a shard and return the result. See examples below.

Example data from `broadcastEval()`:
```javascript
/*
	Example of result of broadcastEval() on a bot with 4 servers split across 2 shards
*/
const res = await client.shard.broadcastEval('this.guilds.map((guild) => guild.members.size)');

console.log(res);
// 	Array: [
//		[	// shard 0
//			30,
//			25
//		],
//		[	// shard 1
//			55,
//			10
//		]
// 	]

````


Example `broadcastEval()` function:
```javascript
/*
  	Example by ZiNc#2032
  	The following code fetches a single guild from across shards
  	NOTE: Fetched guild's properties such as "Guild.members" and "Guild.roles" will not be Managers, but arrays of snowflake IDs
  	
  	discord.js version 11.x
  	client = new discordjs.Client()

  	returns Guild
*/

const getServer = async (guildID) => {
    // broadcast request
    const req = await client.shard.broadcastEval(`this.guilds.get("${guildID}")`);

    // return non-null response or false if not found
    return (req.find((res) => !!res) || false);
}
```
