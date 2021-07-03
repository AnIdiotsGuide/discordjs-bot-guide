---
description: >-
  Using a simple cache to track invites and know which invite was used when a
  new member joins a guild.
---

# Tracking Used Invites

A relatively frequent thing people would love to know, is "what invite someone used to join". Unfortunately, the Discord API does not provide the information about the invite used to join a server.

To get around this, we need to do a couple separate steps. The first one is to go into your bots application page and enable the `GUILD_MEMBERS` privileged intent and add it to your bots client, then the second thing is to fetch all of the invites for each guild and store it in a temporary object. The third is to add, and remove invites from the object via the `inviteCreate` and `inviteDelete` events and finally is to fetch a guild's invite whenever someone joins, and find which one has a new use on it.

Thankfully, that's actually pretty simple!

While the below is a fair approximation of invite tracking, it's still not perfect. As it doesn't track single use invites \(when right-clicking someone, and doing Invite To =&gt; Server\). Those exist only for a few moments and cannot be tracked at all.

## Caching Invites

The first part is to fetch all the invites and keep them cached in an object. This is done in the `ready` event. First, however, we must ensure that the cache is initialized _outside_ of the ready event, be it further up in your main file.

```javascript
// Initialize the invite cache
const invites = {};

// A pretty useful method to create a delay without blocking the whole script.
const wait = require('util').promisify(setTimeout);

client.on('ready', async () => {
  // "ready" isn't really ready. We need to wait a spell.
  await wait(1000);

  // Load all invites for all guilds and save them to the cache.
  client.guilds.cache.forEach(g => {
    g.fetchInvites().then(guildInvites => {
      invites[g.id] = guildInvites;
    });
  });
});
```

## Using the events.

We need to add two events to our code, `inviteCreate` and `inviteDelete`, the code for them are identical, but you can find them below.

```javascript
client.on('inviteCreate', (invite) => {
  // This adds the created invite to the guild object
  invites[invite.guild.id].set(invite.code, invite);
});

client.on('inviteDelete', (invite) => {
  // This deletes the invite from the guild object
  invites[invite.guild.id].delete(invite.code);
});
```

## Catching New Members

So now that we have our `invites` object, we're ready to listen to the `guildMemberAdd` event. When a new member joins, we need to fetch all of the guild's invites once again. Then, we look through our _cached_ invites and see which one has been used, by comparing the current invite's use with the cached ones.

```javascript
client.on('guildMemberAdd', member => {
  // To compare, we need to load the current invite list.
  member.guild.fetchInvites().then(guildInvites => {
    // This is the *existing* invites for the guild.
    const ei = invites[member.guild.id];
    // Update the cached invites for the guild.
    invites[member.guild.id] = guildInvites;
    // Look through the invites, find the one for which the uses went up.
    const invite = guildInvites.find(i => ei.get(i.code).uses < i.uses);
    // This is just to simplify the message being sent below (inviter doesn't have a tag property)
    const inviter = client.users.cache.get(invite.inviter.id);
    // Get the log channel (change to your liking)
    const logChannel = member.guild.channels.cache.find(channel => channel.name === "join-logs");
    // A real basic message with the information we need. 
    logChannel.send(`${member.user.tag} joined using invite code ${invite.code} from ${inviter.tag}. This invite has been used ${invite.uses} times since its creation.`);
  });
});
```

And... well, that's pretty much it. But....

## There's a Catch

So here's the problem. Each time you fetch invites, you're hitting the Discord API with a request for information. While that's not an issue for small bots, it might as the bot grows. I'm not saying that using this code would get you banned from the API - there is no inherent problem with querying the API if it's not abuse. However, there are a few technical issues with performance especially.

The more guilds you have, the more invites are in each guild, the more data you're receiving from the API. Remember, because of the way the `ready` event works, you need to _wait_ a bit before running the fetchInvite method, and the more guilds you have, the more time you need to wait. During this time, member joins won't correctly register because the cache doesn't exist.

Furthermore, the cache itself grows in size, so you're using more memory. On top of which, it takes more time to sort through the invites the more invites exist. It might make the bot appear as being slower to respond to members joining.

So to conclude, the above code works perfectly well and it will not get you in trouble with Discord, but I wouldn't recommend implementing this on larger bots, especially if you're worried about memory and performance.
