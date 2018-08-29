---
description: >-
  Message reactions aren't tracked on uncached messages, but we can get around
  this by using the undocumented "raw" event, which receives all API messages
  sent by Discord.
---

# Raw Events

This will essentially cover how to make use of the `raw` event in order to make use of message reactions. Discord.js processes raw packets in order to give you pretty-formatted events that you make use of in your bots. The underlying architecture, however, is not so pretty. You can find documentation for the general structure of these packets [here](https://discordapp.com/developers/docs/topics/gateway#payloads). The packet contents in `d` can differ very greatly. The contents of that field is each gateway event. For the purpose of this guide, we will take a look at [`messageReactionAdd`](https://discordapp.com/developers/docs/topics/gateway#message-reaction-add).

Here is what a packet can essentially look like:

```javascript
{ t: 'MESSAGE_REACTION_ADD',
  s: 15,
  op: 0,
  d:
   { user_id: '84484653687267328',
     message_id: '447739338655268864',
     emoji: { name: '👌', id: null, animated: false },
     channel_id: '254574550816129024',
     guild_id: '174672265953148929' } }
```

Notice how similar that looks to the structure depicted in the link above this snippet.

You can preview it yourself with the following code:

```javascript
client.on('raw', console.log);
```

Note that this event does get very spammy, as it triggers on everything that happens to your bot, so it's reccommended to play with this on a testing bot. Now, let's say you have the following two events:

```javascript
client.on('messageReactionAdd', (reaction, user) => {
    console.log('a reaction has been added');
});

client.on('messageReactionRemove', (reaction, user) => {
    console.log('a reaction has been removed');
});
```

Let's add a `raw` event listener. Usually the given items are called a packet, so we will call it that for consistency.

```javascript
client.on('raw', packet => {
    // We don't want this to run on unrelated packets
    if (!['MESSAGE_REACTION_ADD', 'MESSAGE_REACTION_REMOVE'].includes(packet.t)) return;
    // Grab the channel to check the message from
    const channel = client.channels.get(packet.d.channel_id);
    // There's no need to emit if the message is cached, because the event will fire anyway for that
    if (channel.messages.has(packet.d.message_id)) return;
    // Since we have confirmed the message is not cached, let's fetch it
    channel.fetchMessage(packet.d.message_id).then(message => {
        // Emojis can have identifiers of name:id format, so we have to account for that case as well
        const emoji = packet.d.emoji.id ? `${packet.d.emoji.name}:${packet.d.emoji.id}` : packet.d.emoji.name;
        // This gives us the reaction we need to emit the event properly, in top of the message object
        const reaction = message.reactions.get(emoji);
        // Check which type of event it is before emitting
        if (packet.t === 'MESSAGE_REACTION_ADD') {
            client.emit('messageReactionAdd', reaction, client.users.get(packet.d.user_id));
        }
        if (packet.t === 'MESSAGE_REACTION_REMOVE') {
            client.emit('messageReactionRemove', reaction, client.users.get(packet.d.user_id));
        }
    });
});
```

Now, both of those events should trigger for any message ever!

