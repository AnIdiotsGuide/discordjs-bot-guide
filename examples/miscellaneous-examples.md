# Miscellaneous Examples

While this page is mostly a dump of the `?examples` command on An Idiot's Guide Official Server, it also serves to put them all in one place, easily accessible for everyone to read.

## Conventions Used in Examples

Conventions are important - they are the agreements on which society functions. So let's take a moment to agree on a few.

### Placeholders

A "Placeholder" is a piece of text that _replaces something else_. In these FAQs we assume the following variables as "placeholders" for your own:

* `client` is a placeholder that corresponds to your `client` variable, as we've covered at the end of the [Getting Started](../getting-started/getting-started-long-version.md) guide. `client.on("ready", () => {` for example.
* `message` is a placeholder corresponds to your `message` event's variable which looks something like this: `client.on("message", msg => {`.

## Examples

### awaitMessages

> Original example by: 🌌 ∫ Lewdcario dx 🐾\#8248

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
    const defaultChannel = guild.channels.find(c=> c.permissionsFor(guild.me).has("SEND_MESSAGES"));
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

> Example by ItsJordan\#4297

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

> Example By Nomsy\#7453

```javascript
// The modules we are using are cheerio, snekfetch, and querystring.
const cheerio = require('cheerio'),
      snekfetch = require('snekfetch'),
      querystring = require('querystring');

// Depending on your command framework (or if you use one), it doesn't have to
// edit messages so you can rework it to fit your needs. Again, this doesn't have
// to be async if you don't care about message editing.
async function googleCommand(msg, args) {

   // These are our two variables. One of them creates a message while we preform a search,
   // the other generates a URL for our crawler.
   let searchMessage = await <Message>.reply('Searching... Sec.');
   let searchUrl = `https://www.google.com/search?q=${encodeURIComponent(msg.content)}`;

   // We will now use snekfetch to crawl Google.com. Snekfetch uses promises so we will
   // utilize that for our try/catch block.
   return snekfetch.get(searchUrl).then((result) => {

      // Cheerio lets us parse the HTML on our google result to grab the URL.
      let $ = cheerio.load(result.text);

      // This is allowing us to grab the URL from within the instance of the page (HTML)
      let googleData = $('.r').first().find('a').first().attr('href');

      // Now that we have our data from Google, we can send it to the channel.
      googleData = querystring.parse(googleData.replace('/url?', ''));
      searchMessage.edit(`Result found!\n${googleData.q}`);

  // If no results are found, we catch it and return 'No results are found!'
  }).catch((err) => {
     searchMessage.edit('No results found!');
  });
}
```

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

```js
client.on('message', async message => {
  const prefixes = ['!', '\\?', '\\/', `<@!?${client.user.id}> `];
  const prefixRegex = new RegExp(`^(${prefixes.join('|')})`);
  const prefix = message.content.match(prefixRegex);
  
  // Go ahead with the rest of your code!
});
```

### Purging a Channel

> Example by eslachance\#4611

Example usage: !purge @user 10 , or !purge 25

```javascript
const user = message.mentions.users.first();
const amount = !!parseInt(message.content.split(' ')[1]) ? parseInt(message.content.split(' ')[1]) : parseInt(message.content.split(' ')[2])
if (!amount) return message.reply('Must specify an amount to delete!');
if (!amount && !user) return message.reply('Must specify a user and amount, or just an amount, of messages to purge!');
message.channel.fetchMessages({
 limit: amount,
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

