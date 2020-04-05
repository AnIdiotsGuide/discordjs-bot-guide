# Miscellaneous Examples

## Conventions Used in Examples

Conventions are important - they are the agreements on which society functions. So let's take a moment to agree on a few.

### Placeholders

A "Placeholder" is a piece of text that _replaces something else_. In these FAQs we assume the following variables as "placeholders" for your own:

* `client` is a placeholder that corresponds to your `client` variable, as we've covered at the end of the [Getting Started](../getting-started/getting-started-long-version.md) guide. `client.on("ready", () => {` for example.
* `message` is a placeholder corresponds to your `message` event's variable which looks something like this: `client.on("message", msg => {`.

## Examples

### awaitMessages

{% hint style="info" %}
Example by: Lewdcario \(84484653687267328\)
{% endhint %}

This example sends a question and waits to receive a message response that says `test`.

```javascript
message.channel.send('What tag would you like to see? This will await will be cancelled in 30 seconds. It will finish when you provide a message that goes through the filter the first time.')
.then(() => {
  message.channel.awaitMessages(response => response.content === 'test', {
    max: 1,
    time: 30000,
    errors: ['time'],
  })
  .then((collected) => {
      message.channel.send(`The collected message was: ${collected.first().content}`);
    })
    .catch(() => {
      message.channel.send('There was no collected message that passed the filter within the time limit!');
    });
});
```

### Creating a guild

Discord quietly changed the Create Guild API endpoint, small bots \(10 guilds or fewer\) are able to create guilds programmatically now. This example will have your bot create a new guild and create a role with the administrator permission, and the single line of code at the bottom will apply it to you when you execute it when you join the guild.

```javascript
/* ES6 Promises */
client.user.createGuild('Example Guild', 'london').then(guild => {
  guild.channels.get(guild.id).createInvite()
    .then(invite => client.users.get('<USERID>').send(invite.url));
  guild.createRole({name:'Example Role', permissions:['ADMINISTRATOR']})
    .then(role => client.users.get('<UserId>').send(role.id))
    .catch(error => console.log(error))
});

/* ES8 async/await */
async function createGuild(client, message) {
  try {
    const guild = await client.user.createGuild('Example Guild', 'london');
    const defaultChannel = guild.channels.find(channel => channel.permissionsFor(guild.me).has("SEND_MESSAGES"));
    const invite = await defaultChannel.createInvite();
    await message.author.send(invite.url);
    const role = await guild.createRole({ name:'Example Role', permissions:['ADMINISTRATOR'] });
    await message.author.send(role.id);
  } catch (e) {
    console.error(e);
  }
}
createGuild(client, message);
// Run this once you've joined the bot created guild.
message.member.addRole('<THE ROLE ID YOU GET SENT>');
```

### Command Cooldown

{% hint style="info" %}
Example by ItsJordan\#4297
{% endhint %}

Adds a cooldown to your commands so the user will have to wait 2.5 seconds between each command.

You can change the nature of the cool down by changing the return to something else.

```javascript
// First, this must be at the top level of your code, **NOT** in any event!
const talkedRecently = new Set();
```

```javascript
// Inside your message event, this code will stop any command during cooldown.
// Should be placed after your code that checks for bots & prefix, for best performance

if (talkedRecently.has(message.author.id))
  return;

// Adds the user to the set so that they can't talk for 2.5 seconds
talkedRecently.add(message.author.id);
setTimeout(() => {
  // Removes the user from the set after 2.5 seconds
  talkedRecently.delete(message.author.id);
}, 2500);
```

### Google Search Command

{% hint style="info" %}
This example was removed as Google keeps changing their page to prevent scraping. Google doesn't have a search API, and scraping their page for results is against their TOS.
{% endhint %}

### Mention Prefix

Requiring a little bit of regex, this will catch when a message starts with the bot being mentioned.

```javascript
client.on('message', message => {
  const prefixMention = new RegExp(`^<@!?${client.user.id}> `);
    const prefix = message.content.match(prefixMention) ? message.content.match(prefixMention)[0] : '!';

  // Go ahead with the rest of your code!
});
```

### Multiple Prefixes

Let's make it 3 prefixes, this is fairly universal. This could also be in the config.json once you get there. So we'll start by setting it to false, we'll overwrite this in the loop. We should loop through the array using _for...of_ which is cleaner than that damn _i_ counter loop. This makes the prefix variable something else than false \('truthy'\) if the message starts with the prefix. If the message doesn't start with any of the 3 prefixes, then we can simply exit as usual.

```javascript
client.on("message", message => {
  const prefixes = ['!', '?', '/'];
  let prefix = false;
  for(const thisPrefix of prefixes) {
    if(message.content.startsWith(thisPrefix)) prefix = thisPrefix;
  }
  if(!prefix) return;

  // Go ahead with the rest of your code!
});
```

This allows you to include the mention as a prefix, on top of the previous example.

### Multiple Prefixes Extension

```javascript
client.on('message', async message => {
  const prefixes = ['!', '\\?', '\\/', `<@!?${client.user.id}> `];
  const prefixRegex = new RegExp(`^(${prefixes.join('|')})`);
  const prefix = message.content.match(prefixRegex);

  // Go ahead with the rest of your code!
});
```

### Purging a Channel

{% hint style="info" %}
Example by Hindsight \(139412744439988224\)
{% endhint %}

Example usage: !purge @user 10 , or !purge 25

```javascript
const user = message.mentions.users.first();
// Parse Amount
const amount = !!parseInt(message.content.split(' ')[1]) ? parseInt(message.content.split(' ')[1]) : parseInt(message.content.split(' ')[2])
if (!amount) return message.reply('Must specify an amount to delete!');
if (!amount && !user) return message.reply('Must specify a user and amount, or just an amount, of messages to purge!');
// Fetch 100 messages (will be filtered and lowered up to max amount requested)
message.channel.fetchMessages({
 limit: 100,
}).then((messages) => {
 if (user) {
 const filterBy = user ? user.id : Client.user.id;
 messages = messages.filter(m => m.author.id === filterBy).array().slice(0, amount);
 }
 message.channel.bulkDelete(messages).catch(error => console.log(error.stack));
});
```

### Swear Detector

This quick & dirty swear detector takes an array of swear words we don't want to see, and triggers on it.

```javascript
const swearWords = ["darn", "shucks", "frak", "shite"];
if( swearWords.some(word => message.content.includes(word)) ) {
  message.reply("Oh no you said a bad word!!!");
  // Or just do message.delete();
}
```

### Kicking users \(or bots\) from a voice channel

Support for kicking members from voice channels has now been added by Discord and can be achieved by doing the following.

```javascript
// Make sure the bot user has permissions to move members in the guild:
if (!message.guild.me.hasPermission('MOVE_MEMBERS')) return message.reply('Missing the required `Move Members` permission.');

// Get the mentioned user/bot and check if they're in a voice channel:
const member = message.mentions.members.first();
if (!member) return message.reply('You need to @mention a user/bot to kick from the voice channel.');
if (!member.voiceChannel) return message.reply('That user/bot isn\'t in a voice channel.');

// Now we set the member's voice channel to null, in other words disconnecting them from the voice channel.
member.setVoiceChannel(null);

// Finally, pass some user response to show it all worked out:
message.react('👌');
/* or just "message.reply", etc.. up to you! */
```

This does the same as clicking the disconnect button on a user or bot while they are in a voice channel.

