---
description: >-
  Using a simple cache to track invites and know which invite was used when a
  new member joins a guild.
---

# Tracking Used Invites

A relatively frequent thing people would love to know, is "what invite someone used to join". Unfortunately, the Discord API does not provide the information about the invite used to join a server.

To get around this, we need to do two separate steps. The first one is to fetch all of the invites for each guild and store it in a temporary [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map). The second is to fetch a guild's invite whenever someone joins, and find which one has a new use on it. Thankfully, that's actually pretty simple!

{% hint style="info" %}
For this code to work, you need to have your privileged `GUILD_MEMBERS` intent enabled in your Bot's Application Page. You also need to make sure to initialize that the Intents property in your Client Options has this intent mentioned. These two vital steps will give you access to the privileged `guildMemberAdd` and `guildMemberRemove` events.
{% endhint %}

While the below is a fair approximation of invite tracking, it's still not perfect. There are 2 things it doesn't track:

* Temporary one-use invites \(when right-clicking someone, and doing Invite To =&gt; Server\). Those exist only for a few moments and cannot be tracked at all.

## Caching Invites

The first part is to fetch all the invites and keep them cached in an Map. This is done in the `ready` event. First, however, we must ensure that the cache is initialized _outside_ of the ready event.

```javascript
// Initialize the invite cache
const invites = new Map();

// A pretty useful method to create a delay without blocking the whole script.
const wait = require("timers/promises").setTimeout;

client.on("ready", async () => {
  // "ready" isn't really ready. We need to wait a spell.
  await wait(1000);

  // Loop over all the guilds
  client.guilds.cache.forEach(async (guild) => {
    // Fetch all Guild Invites
    const firstInvites = await guild.invites.fetch();
    // Set the key as Guild ID, and create a map which has the invite code, and the number of uses
    invites.set(guild.id, new Map(firstInvites.map((invite) => [invite.code, invite.uses])));
  });
});
```

Moving on to the second part, we need to listen to the `inviteCreate` and `inviteDelete` event provided by the API, and update our cache accordingly.

```javascript
client.on("inviteDelete", (invite) => {
  // Delete the Invite from Cache
  invites.get(invite.guild.id).delete(invite.code);
});

client.on("inviteCreate", (invite) => {
  // Update cache on new invites
  invites.get(invite.guild.id).set(invite.code, invite.uses);
});

```

## Caching Guilds that have been added/removed

Since we already have the Guilds and their Invites cached, we need to check if we've been added or removed from any of the Guilds. If we're added, we need to fetch and cache all invites. If we're removed, we can just delete the data from our cache. It is as simple as this

```javascript
client.on("guildCreate", (guild) => {
  // We've been added to a new Guild. Let's fetch all the invites, and save it to our cache
  guild.invites.fetch().then(guildInvites => {
    // This is the same as the ready event
    invites.set(guild.id, new Map(guildInvites.map((invite) => [invite.code, invite.uses])));
  })
});

client.on("guildDelete", (guild) => {
  // We've been removed from a Guild. Let's delete all their invites
  invites.delete(guild.id);
});
```

## Catching New Members

So now that we have our `invites` object, we're ready to listen to the `guildMemberAdd` event. When a new member joins, we need to fetch all of the guild's invites once again. Then, we look through our _cached_ invites and see which one has been used, by comparing the current invite's use with the cached ones.

```javascript
client.on("guildMemberAdd", member => {
  // To compare, we need to load the current invite list.
  member.guild.invites.fetch().then(newInvites => {
    // This is the *existing* invites for the guild.
    const oldInvites = invites.get(member.guild.id);
    // Look through the invites, find the one for which the uses went up.
    const invite = newInvites.find(i => i.uses > oldInvites.get(i.code));
    // This is just to simplify the message being sent below (inviter doesn't have a tag property)
    const inviter = client.users.cache.get(invite.inviter.id);
    // Get the log channel (change to your liking)
    const logChannel = member.guild.channels.cache.find(channel => channel.name === "join-logs");
    // A real basic message with the information we need. 
    inviter
      ? logChannel.send(`${member.user.tag} joined using invite code ${invite.code} from ${inviter.tag}. Invite was used ${invite.uses} times since its creation.`)
      : logChannel.send(`${member.user.tag} joined but I couldn't find through which invite.`);
  });
});
```

And... well, that's pretty much it. But....

## There's a Catch

So here's the problem. Each time you fetch invites, you're hitting the Discord API with a request for information. While that's not an issue for small bots, it might as the bot grows. I'm not saying that using this code would get you banned from the API - there is no inherent problem with querying the API if it's not abuse. However, there are a few technical issues with performance especially.

The more guilds you have, the more invites are in each guild, the more data you're receiving from the API. Remember, because of the way the `ready` event works, you need to _wait_ a bit before running the fetch method, and the more guilds you have, the more time you need to wait. During this time, member joins won't correctly register because the cache doesn't exist.

Furthermore, the cache itself grows in size, so you're using more memory. On top of which, it takes more time to sort through the invites the more invites exist. It might make the bot appear as being slower to respond to members joining.

So to conclude, the above code works perfectly well and it _should_ not get you in trouble with Discord, but I wouldn't recommend implementing this on larger bots, especially if you're worried about memory and performance.

