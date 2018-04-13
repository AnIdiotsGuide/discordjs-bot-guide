# Starboard

This is a long awaited feature, requested by many people.

Let's begin by talking about what a starboard is. This is an example taken from the Discord.js Official Server.

![Starboard](/assets/starboard.png)

> Before we start, there is one thing you need to now. This code is compatible with only Guidebot Class. This is due to a restriction with passing `client` to the messageReactionAdd and messageReactionRemove events.

So, let's begin!

In this block, we just do some simple setup for later on. For ease, I personally define the message object as `message`, but this is completely optional. Next, we grab the starboardChannel key from the guilds settings. 

Then, we preform a couple of checks on the reaction and the message. First, we check if the reaction is **NOT** the unicode star emote. Next, we preform two checks on the message, checking to see if the user who added the reaction authored the message, and if the message author is a bot. If neither of these checks return true, we're good to move on.

```js
module.exports = class {
  constructor(client) {
    this.client = client;
  }

async run(reaction, user) {
  const message = reaction.message;
  if (reaction.emoji.name !== '⭐') return;
  if (message.author.id === user.id) return message.channel.send(`${user}, you cannot star your own messages.`);
  if (message.author.bot) return message.channel.send(`${user}, you cannot star bot messages.`);
  const { starboardChannel } = this.client.settings.get(message.guild.id);
  }
}
```

Now, this next block may look real complex, but it really isn't. Let's break it down.

First, we declare 6 variables here. `fetch` will fetch the last 100 messages in the starboard channel, `stars` attempts to find an embed who's footer ends with the message's id.

Next, we'll start an if statement for if there is already an embed in the starboard channel for this message, and we also define our final 4 variables. `star` is a simple regex to check how many stars the embed already has, `foundStar` allows us to use the color of the pre-existing embed, `image` checks if there is anything attached to the message, and embed is our new embed!

I told you it wasn't that complicated. Let's keep going.

```js
const fetch = await message.guild.channels.find('name', starboardChannel).fetchMessages({ limit: 100 });
const stars = fetch.find(m => m.embeds[0].footer.text.startsWith('⭐') && m.embeds[0].footer.text.endsWith(message.id));
if (stars) {
  const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text);
  const foundStar = stars.embeds[0];
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : '';
  const embed = new RichEmbed()
    .setColor(foundStar.color)
    .setDescription(foundStar.description)
    .setAuthor(message.author.name, message.author.displayAvatarURL)
    .setTimestamp()
    .setFooter(`⭐ ${parseInt(star[1])+1} | ${message.id}`)
    .setImage(image);
  const starMsg = await message.guild.channels.find('name', starboard).fetchMessage(stars.id);
  await starMsg.edit({ embed });
}
```

Now, if you were to just use the code above, your starboard would only function if there was already a message in the starboard channel. Let's take care of that.

Here we add an if statement that mimics the previous block, but this time manually setting the color of the embed, and also manually setting the amount of stars the embed will have.

```js
if (!stars) {
  if (!message.guild.channels.exists('name', starboard)) throw `It appears that you do not have a \`${starboard}\` channel.`;
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : '';
  if (image === '' && message.cleanContent.length < 1) return message.channel.send(`${user}, you cannot star an empty message.`);
  const embed = new RichEmbed()
    .setColor(15844367)
    .setDescription(message.cleanContent)
    .setAuthor(message.author.tag, message.author.displayAvatarURL)
    .setTimestamp(new Date())
    .setFooter(`⭐ 1 | ${message.id}`)
    .setImage(image);
  await message.guild.channels.find('name', starboard).send({ embed });
}
```

Now, if you've been following along exactly, you have a working starboard! But if you like a TL;DR, here ya go. 

```js
module.exports = class {
  constructor(client) {
    this.client = client;
  }

  async run(reaction, user) {
    const message = reaction.message;
    if (reaction.emoji.name !== '⭐') return;
    if (message.author.id === user.id) return message.channel.send(`${user}, you cannot star your own messages.`);
    if (message.author.bot) return message.channel.send(`${user}, you cannot star bot messages.`);
    const { starboardChannel } = this.client.settings.get(message.guild.id);
    const fetch = await message.guild.channels.find('name', starboard).fetchMessages({ limit: 100 });
    const stars = fetch.find(m => m.embeds[0].footer.text.startsWith('⭐') && m.embeds[0].footer.text.endsWith(message.id));
    if (stars) {
      const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text);
      const foundStar = stars.embeds[0];
      const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : '';
      const embed = new RichEmbed()
        .setColor(foundStar.color)
        .setDescription(foundStar.description)
        .setAuthor(message.author.name, message.author.displayAvatarURL)
        .setTimestamp()
        .setFooter(`⭐ ${parseInt(star[1])+1} | ${message.id}`)
        .setImage(image);
      const starMsg = await message.guild.channels.find('name', starboard).fetchMessage(stars.id);
      await starMsg.edit({ embed });
    }
    if (!stars) {
      if (!message.guild.channels.exists('name', starboard)) throw `It appears that you do not have a \`${starboard}\` channel.`;
      const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : '';
      if (image === '' && message.cleanContent.length < 1) return message.channel.send(`${user}, you cannot star an empty message.`);
      const embed = new RichEmbed()
        .setColor(15844367)
        .setDescription(message.cleanContent)
        .setAuthor(message.author.tag, message.author.displayAvatarURL)
        .setTimestamp(new Date())
        .setFooter(`⭐ 1 | ${message.id}`)
        .setImage(image);
      await message.guild.channels.find('name', starboard).send({ embed });
    }
  }

  extension(reaction, attachment) {
    const imageLink = attachment.split('.');
    const typeOfImage = imageLink[imageLink.length - 1];
    const image = /(jpg|jpeg|png|gif)/gi.test(typeOfImage);
    if (!image) return '';
    return attachment;
  }
};
```

So, all that code up there is only for if a reaction is added. Now, we'll handle if a reaction is removed.

Here, we very closely mimic the messageReactionAdd event, only this time we'll **subtract** 1 from the total amount of stars that are on the embed.

```js
if (stars) {
  const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text);
  const foundStar = stars.embeds[0];
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : '';
  const embed = new RichEmbed()
    .setColor(foundStar.color)
    .setDescription(foundStar.description)
    .setAuthor(message.author.name, message.author.displayAvatarURL)
    .setTimestamp()
    .setFooter(`⭐ ${parseInt(star[1])-1} | ${message.id}`)
    .setImage(image);
  const starMsg = await message.guild.channels.find('name', starboardChannel).fetchMessage(stars.id);
  await starMsg.edit({ embed });
}
```