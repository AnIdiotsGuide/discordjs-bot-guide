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
    // https://discord.js.org/#/docs/main/v11/class/ShardingManager

    // 'auto' handles shard count automatically
    totalShards: 'auto', 

    // your bot token
    token: 'YOUR_TOKEN_GOES_HERE'
});

// Spawn your shards
manager.spawn();

// The shardCreate event is emitted when a shard is created.
// You can use it for something like logging shard launches.
manager.on('shardCreate', (shard) => console.log(`Shard ${shard.id} launched`));
```

## Sharing Information Between Shards

{% hint style="info" %}
Information is not readily available between shards. In order to get or share information across shards, you will need to make use of either `fetchClientValues()` or `broadcastEval()`.
{% endhint %}

Remember how we were talking about sharding being a method of "splitting" the bot into multiple instances of itself? Because your sharded bot is now in separate, individual instances, things like your adding your total guilds or getting a specific guild are not as simple as they were before. We must now use either [`fetchClientValues`](sharding.md##FetchClientValues) or [`broadcastEval`](sharding.md##BroadcastEval) to get information from across shards.

These two functions are your go-to for getting any information from other shards, so get familiar with them!

## FetchClientValues

[`fetchClientValues`](https://discord.js.org/#/docs/main/v11/class/ShardClientUtil?scrollTo=fetchClientValues) gets Client properties from all shards. This is what you should use when you would like to get any of the nested properties of the Client, such as `guilds.size` or `uptime`. It's useful for getting things like Collection sizes, basic client properties, and unprocessed information about the client.

Example:

```javascript
/*
    Example result of fetchClientValues() on a bot with 4,300 guilds split across 4 shards.
    Assume this is being executed on shard 0, the first shard.
*/

// If we just get our client.guilds.size, it will return
// only the number of guilds on the shard this is being run on.
console.log('client.guilds.size');
// 1050

// If we would like to get our client.guilds.size from all
// of our shards, we must make use of fetchClientValues().
const res = await client.shard.fetchClientValues('guilds.size');
console.log(res);
//     [
//        1050,    // shard 0
//        1100,    // shard 1
//        1075,    // shard 2
//        1075    // shard 3
//    ]

`
```

Let's say you want to do something like get your total server count - In a non-sharded environment, this would be as simple as getting the `client.guilds.size`. However in a sharded environment, `client.guilds.size` will return not the total servers your bot is in. Instead it returns only the total number of servers _on this shard_, like in the first part of the example above.

Here's an example of a function that uses `fetchClientValues()` to first get, then add the total number of guilds from _all shards_ \(i.e. your bot's total guild count\):

```javascript
/*
      Example by ZiNc#2032
      The following code fetches total combined shards' server counts.

      discord.js version 11.x
      client = new discordjs.Client()

      returns number
*/

const getServerCount = async () => {
    // get guild collection size from all the shards
    const req = await client.shard.fetchClientValues('guilds.size');

    // return the added value
    return req.reduce((p, n) => p + n, 0);
}
```

{% hint style="info" %}
`fetchClientValues()` does not allow you to make use of javascript methods or client methods to get or process information before returning it. It only allows you to get information from client properties.
{% endhint %}

## BroadcastEval

[`broadcastEval`](https://discord.js.org/#/docs/main/v11/class/ShardClientUtil?scrollTo=broadcastEval) evaluates the input _in the context of each shard's Client\(s\)_ \(i.e. `this` is used to reference the `Client`\). This is what you should use when you want to execute a method or process data on a shard and return the result. It's useful for getting information that isn't available through client properties and must instead be retrieved through the use of methods.

Example:

```javascript
/*
    Example of result of broadcastEval() on a bot with 4 servers split across 2 shards.
    Assume this is being executed on shard 0, the first shard.
*/

// If we just map our guilds' members.size, it will return
// only the mapped members.size of the shard this is being run on.
console.log(client.guilds.map((guild) => guild.members.size));
//        [
//            30,
//            25
//        ],

// If we would like to map our guilds' members.size from our
// servers on all of our shards, we must make use of broadcastEval().
// Remember, this runs in the context of the client, so we refer to the
// Client using "this".
const res = await client.shard.broadcastEval('this.guilds.map((guild) => guild.members.size)');
console.log(res);
//     [
//        [    // shard 0
//            30,
//            25
//        ],
//        [    // shard 1
//            55,
//            10
//        ]
//     ]

`
```

Say you want to get a guild from your client. In a non-sharded environment, you would simply use `client.guilds.get('ID')` or something of that nature and then carry on with your code. In a sharded environment however, it is possible that the guild you're trying to get _is not present on the shard_. In order to get the guild for use, you would then need to fetch it from whatever shard it is present on using `broadcastEval()`.

Here's an example of a function that uses `broadcastEval()` to get a single guild no matter what shard it is present on:

```javascript
/*
      Example by ZiNc#2032
      The following code fetches a single guild from across shards.

    NOTE: Fetched guild's properties such as "Guild.members" and "Guild.roles" will
    not be Managers; these properties will be arrays of snowflake IDs.

      discord.js version 11.x
      client = new discordjs.Client()

      returns Guild
*/

const getServer = async (guildID) => {
    // try to get guild from all the shards
    const req = await client.shard.broadcastEval(`this.guilds.get("${guildID}")`);

    // return non-null response or false if not found
    return (req.find((res) => !!res) || false);
}
```

{% hint style="info" %}
`broadcastEval()` will only return basic javascript objects, and will not return Discord.js structures such as Collections, Guild objects, Member object, and the likes. You will need to interact directly with the API or build these structures yourself after getting a response back from your `broadcastEval()`.
{% endhint %}

