# Sharding

**Sharding** is the method by which a bot's code is "split" into multiple _instances_ of itself. When a bot is sharded, each shard handles only a certain percentage of all the guilds the bot is on.

There are additional difficulties when sharding a bot that add complexity to your code \(one of the reasons you shouldn't shard too early\). You do not need to worry about sharding until your bot hits around 2,400 guilds. YOU MUST SHARD by the time you hit 2,500 guilds, however.

## Sharding Styles

There are two styles of sharding that we'll be discussing: [internal sharding](sharding.md#internal-sharding) and [traditional sharding](sharding.md#traditional-sharding). Each of these sharding styles holds benefits depending on your situation.

## Internal Sharding

`internal` sharding is the method by which a bot's code creates multiple shard connections to the Discord API _within a single process_. This means that all the guilds, channels, and users on one shard will be available to another shard via a direct call \(e.g. `client.guilds.cache.get('GUILD_ID')`\). Due to the large memory size the single bot process will grow to using this style of sharding, it is not ideal for bots with many guilds.

### Internally Sharded Client

If you would like to use this, adjust the Client options in your main bot file where you define your client like so:

```javascript
/*
    The following code goes into your main bot file.
*/

// Include discord.js ShardingManager
const { Client } = require("discord.js");

// When we define our client, we include the property "shardCount"
// and set it to 'auto' to allow the client to automatically create
// the correct number of shards.
// If you would like to have a different number of shards, you may
// also set this to a number.
const client = new Client({ shardCount: "auto" });
```

## Traditional Sharding

`traditional` sharding is the method by which a bot's code spawns individual child processes via a main shard manager process, each child process being one shard of the bot. When using this style of sharding, guilds, channels, and users on one shard will _not_ be available to another via direct call \(e.g. `client.guilds.cache.get('GUILD_ID')`\) because each shard is in a separate process.

This style of sharding is ideal for larger bots, or bots that need to be scalable to allow for future growth. The rest of this page will discuss [how to make use of traditional sharding](sharding.md#example-sharding-manager-code) and [how to share information between shards](sharding.md#sharing-information-between-shards).

To learn how to make use of this, read on!

### Traditional Sharding Caveats

* Collections do not cache data from all shards, so you can't grab data from a guild in another shard easily.
* In order to do anything across shards you need to worry about using [`fetchClientValues`](sharding.md#fetchclientvalues) and [`broadcastEval`](sharding.md#broadcasteval) \(Examples and explanation below\).

  And again:

* Traditionally sharded bots often gain very marginal performance increase and might even use _more memory_ due to using more node processes.
* If you're using any sort of database or connection, multiple shards may cause issues with multiple processes connecting to a single end point.

### Example Sharding Manager Code

```javascript
/*
    The following code goes into it's own file, and you run this file
    instead of your main bot file.
*/

// Include discord.js ShardingManager
const { ShardingManager } = require("discord.js");

// Create your ShardingManager instance
const manager = new ShardingManager("./YOUR_BOT_FILE_NAME.js", {
    // for ShardingManager options see:
    // https://discord.js.org/#/docs/main/stable/class/ShardingManager
    totalShards: "auto",
    token: "YOUR_TOKEN_GOES_HERE"
});

// Emitted when a shard is created
manager.on("shardCreate", (shard) => console.log(`Shard ${shard.id} launched`));

// Spawn your shards
manager.spawn();
```

## Sharing Information Between Shards

Remember how we were talking about sharding being a method of "splitting" the bot into multiple instances of itself? Because of this, things like your adding your total guilds or getting a specific guild are not as simple as they were before. We must now use either [`fetchClientValues`](sharding.md#fetchclientvalues) or [`broadcastEval`](sharding.md#broadcasteval) to get information from across shards.

These two functions are your go-to for getting any information from other shards, so get familiar with them!

### FetchClientValues

[`fetchClientValues`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=fetchClientValues) gets Client properties from all shards. This is what you should use when you would like to get any of the nested properties of the Client, such as `guilds.cache.size` or `uptime`. It's useful for getting things like Collection sizes, basic client properties, and unprocessed information about the client.

Example:

```javascript
/*
    Example result of fetchClientValues() on a bot with 4,300 guilds split across
    4 shards.
    Assume this is being executed on shard 0, the first shard.
*/

// If we just get our client.guilds.cache.size, it will return
// only the number of guilds on the shard this is being run on.
console.log(client.guilds.cache.size);
// 1050

// If we would like to get our client.guilds.cache.size from all
// of our shards, we must make use of fetchClientValues().
const res = await client.shard.fetchClientValues("guilds.cache.size");
console.log(res);
//     [
//        1050,    // shard 0
//        1100,    // shard 1
//        1075,    // shard 2
//        1075     // shard 3
//    ]
```

Let's say you want to do something like get your total server count - In a non-sharded environment, this would be as simple as getting the `client.guilds.cache.size`. However, in this case `client.guilds.cache.size` will not return the total servers your bot is in. Instead it returns only the total number of servers _on this shard_, like in the first part of the example above.

Here's an example of a function that uses `fetchClientValues()` to first get, then add the total number of guilds from _all shards_ \(i.e. your bot's total guild count\):

```javascript
/*
    Example by ZiNc#2032

      discord.js version 12.x
      client = new discordjs.Client()

      returns number
*/

const getServerCount = async () => {
    // get guild collection size from all the shards
    const req = await client.shard.fetchClientValues("guilds.cache.size");

    // return the added value
    return req.reduce((p, n) => p + n, 0);
}
```

{% hint style="info" %}
`fetchClientValues()` does not allow you to make use of javascript methods or client methods to get or process information before returning it. It only allows you to get information from client properties.
{% endhint %}

### BroadcastEval

[`broadcastEval`](https://discord.js.org/#/docs/main/stable/class/ShardClientUtil?scrollTo=broadcastEval) evaluates the input in the context of each shard's Client\(s\). This is what you should use when you want to execute a method or process data on a shard and return the result. It's useful for getting information that isn't available through client properties and must instead be retrieved through the use of methods.

Example:

```javascript
/*
    Example of result of broadcastEval() on a bot with 4 servers split across
    2 shards.
    Assume this is being executed on shard 0, the first shard.
*/

// If we just map our guilds' members.cache.size, it will return
// only the mapped members.cache.size of the shard this is being run on.
console.log(client.guilds.cache.map((guild) => guild.members.cache.size));
//        [
//            30,
//            25
//        ],

// If we would like to map our guilds' members.cache.size from our
// servers on all of our shards, we must make use of broadcastEval().
// Remember, this runs in the context of the client, so we refer to the
// Client using "this".
const res = await client.shard.broadcastEval((c) => c.guilds.cache.map((guild) => 
    guild.members.cache.size));
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
```

Say you want to get a guild from your client. In a non-sharded environment, you would simply use `client.guilds.cache.get('ID')` or something of that nature and then carry on with your code. In this case however, it is possible that the guild you're trying to get _is not present on the shard_. In order to get the guild for use, you would then need to fetch it from whatever shard it is present on using `broadcastEval()`.

Here's an example of a function that uses `broadcastEval()` to get a single guild no matter what shard it is present on:

```javascript
/*
      Example by ZiNc#2032

    NOTE: Fetched guild's properties such as "Guild.members.cache" and
    "Guild.roles.cache" will not be Managers or Collections; these
    properties will be arrays of snowflake IDs.

      discord.js version 12.x
      client = new discordjs.Client()

      returns Guild
*/

const getServer = async (guildID) => {
    // try to get guild from all the shards
    const req = await client.shard.broadcastEval((c, id) => c.guilds.cache.get(id), { 
        context: guildID
    });

    // return Guild or null if not found
    return req.find(res => !!res) || null;
}
```

{% hint style="info" %}
`broadcastEval()` will only return basic javascript objects, and will not return Discord.js structures such as Collections, Guild objects, Member object, and the likes. You will need to interact directly with the API or build these structures yourself after getting a response back from your `broadcastEval()`.
{% endhint %}

Example of a Guild object returned by [`broadcastEval`](sharding.md#broadcasteval):

```javascript
const res = await client.shard.broadcastEval((c) => c.guilds.cache.map((guild) => 
    guild.members.cache.size));
console.log(res);
//     [
//        [    // whichever shard has the guild
//          {
//            members: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            channels: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            roles: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            emojis: [
//              '123456789123456789', '123456789123456789', '123456789123456789',
//              '123456789123456789', '123456789123456789', '123456789123456789'
//            ],
//            afkChannelID: null,
//            afkTimeout: 300,
//            applicationID: null,
//            banner: null,
//            bannerURL: null
//            createdTimestamp: 1234567891234,
//            defaultMessageNotifications: 'MENTIONS',
//            deleted: false,
//            description: null,
//            explicitContentFilter: 'MEMBERS_WITHOUT_ROLES',
//            features: [ 'ANIMATED_ICON', 'INVITE_SPLASH' ],
//            icon: 'ICON_ID',
//            iconURL: 'ICON_URL',
//            id: '123456789123456789',
//            joinedTimestamp: 1234567891234,
//            large: true,
//            memberCount: 6000,
//            mfaLevel: 1,
//            name: 'Server_Name',
//            nameAcronym: 'E',
//            ownerID: '123456789123456789',
//            premiumSubscriptionCount: 3,
//            premiumTier: 1,
//            publicUpdatesChannelID: null,
//            region: 'us-east',
//            rulesChannelID: null,
//            shardID: 0,
//            splash: 'SPLASH_ID',
//            splashURL: 'SPLASH_URL',
//            systemChannelFlags: 1,
//            systemChannelID: '123456789123456789',
//            vanityURLCode: null,
//            verificationLevel: 'MEDIUM',
//          }
//        ],
//
//      ...null // all other shard replies
//     ]
```
