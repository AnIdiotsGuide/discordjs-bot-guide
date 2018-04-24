# Raw Events

Raw events are used for uncached messages. How would we use raw events? We can use them to check when a message has been edited, reaction was added or removed etc. So, with that being said, let's get started!

> Please keep in mind that we are using discord@11.3.2 for this guide.

So, how would we use said event? We can initialize it like we would with any event!

```js
const Discord = require('discord.js')
const client = new Discord.Client()
client.on('raw', data => {
  // We will get into more here
})
```

This is initializing the `raw` event and we can do things with `data`. "What king of data does `data` provide us?" Below is an example of a message being edited using the data.

```js
{ t: 'MESSAGE_UPDATE',
  s: 611,
  op: 0,
  d: 
  // Depending on the message that is being edited, you will get different sets of data.
   { id: '438455591921254401',
     embeds: [ [Object] ],
     channel_id: '265156361791209475',
     guild_id: '264445053596991498' 
   }
}
```
> *Not all events can be used with the raw events. Example: MESSAGE DELETE will not work as the message is not cached into the bot, so we cannot get said message.*

So, what does this data mean, you may ask. Well, the `t` stands for 'type' and the `d` stands for 'data'. So, what can we do with this information? Lets log `edited` messages, shall we?

```js
const Discord = require('discord.js')
const client = new Discord.Client()
client.on('raw', async (data) => {
  //We want to establish the type, we are looking for "MESSAGE_UPDATE"
  if (data.t !== 'MESSAGE_UPDATE') return
  
  // Before logging, we need to establish a channel, through the guild
  const logs = client.guilds.get(data.d.guild_id).channels.find('name', 'logs')
  
  // If we cannot find the logs, log it in the console
  if (!logs) return console.log('Cannot find a \'logs\' channel.')
  
  // Now lets send the information!
  const msg2send = await logs.send('Fetching information...')
  
})
```

I would like to take a second and explain something. Depending on the event, let it be `MESSAGE_UPDATE` in our case, we do get a user collection. This is what it looks like down below:

```js
// All of this information is stored under `data`
 d: 
   { type: 0,
     tts: false,
     timestamp: '2018-04-24T21:58:36.561000+00:00',
     pinned: false,
     nonce: null,
     mentions: [],
     mention_roles: [],
     mention_everyone: false,
     id: '438458768418668544',
     embeds: [],
     edited_timestamp: '2018-04-24T21:58:41.976538+00:00',
     content: 'test',
     channel_id: '423299411599294464',
     author: 
      { username: 'Zoth The Wumpus',
        id: '286270602820452353',
        discriminator: '6066',
        avatar: '57361ef0f8e23e02a44069c3dd5f5f47' },
     attachments: [],
     guild_id: '366705297013735425' } }
```

Now, lets edit our message

```js
    // We want to see if we do have a username
    const username = data.d.author
    
    // Of course we return if the user cannot be found (Yes this is possible for some reason)
    if (!username) return await msg2send.edit('There was an error fetching the username.')
    
    // Now we can update the message!
    await msg2send.edit(`${username.username} has edited a message in ${client.guilds.get(data.d.guild_id).channels.get(data.d.channel_id).name}`)
 ```
And there you have it! A simple guide to using raw events. Now you can make an effecient starboard using them.<br>

For those of you who like to view the finalized product, here you go!

```js
const Discord = require('discord.js')
const client = new Discord.Client()
client.on('raw', async (data) => {
  if (data.t !== 'MESSAGE_UPDATE') return
  const logs = client.guilds.get(data.d.guild_id).channels.find('name', 'logs')
  if (!logs) return console.log('Cannot find a \'logs\' channel.')
  const msg2send = await logs.send('Fetching information...')
  const username = data.d.author 
  if (!username) return await msg2send.edit('There was an error fetching the author.')
  await msg2send.edit(`${username.username} has edited a message in ${client.guilds.get(data.d.guild_id).channels.get(data.d.channel_id).name}`)
})
```
    
