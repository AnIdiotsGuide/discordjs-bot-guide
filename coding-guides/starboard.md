# Starboard

This is a long awaited feature, requested by many people.

Let's begin by talking about what a starboard is. This is an example taken from the Discord.js Official Server.

![Starboard](/assets/starboard.png)

> Before we start, there is one thing you need to now. This code is compatible with only Guidebot Class. This is due to a restriction with passing `client` to the messageReactionAdd and messageReactionRemove events.

So, let's begin!

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

This is some simple setup for later on. For ease, I define message as `message`, but this is completely optional. Next, we grab the starboardChannel key from the guilds settings. Then, we preform a couple of checks on the reaction and the message. First, we check if the reaction is the unicode star emote. Then, we preform two checks on the message, which are checking if the user who added the star is the author of the message, and then we check if the author of the message is a bot. If neither of these checks return true, then we're good to go. Let's move on.

```js
async run(reaction, user) {
  const message = reaction.message;
  if (reaction.emoji.name !== '⭐') return;
  if (message.author.id === user.id) return message.channel.send(`${user}, you cannot star your own messages.`);
  if (message.author.bot) return message.channel.send(`${user}, you cannot star bot messages.`);
  const { starboardChannel } = this.client.settings.get(message.guild.id)
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
}
```

This may look real complex, but it really isn't. Let's break it down.

You might first notice that we declare 6 variables here. `fetch` will fetch the last 100 messages in the starboard channel. Next, `stars` attempts to find an embed who's footer ends with the message's id. Here's where the fun starts. As you've probably noticed, we start an if statement here. This is where the fun begins.

If a message is found within the stars collection, we gather how many stars the message currently has. Next, we detect whether or not the message has an attachment. After that, we create a new Rich Embed and edit the embed, adding one to the star count, passing all of the required params to the function. Next, we get the message id of the embed in the starboard channel, and edit it.

You might notice that the last block doesn't account for if there's no embed already in the star channel, so let's take care of that.

```js
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
```

Now we account for if there's no message in the starboard for that message. We do exactly the same thing we did in the other block, only changing a few things this time around. We check to see if a starboard channel exists, and we create the embed with one star.

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

So, all that code up there is only for if a reaction is added. Now, we have to handle if a reaction is removed. The code below is a modified version of the code above, and should be fairly easy for you to figure out. 

```js
module.exports = class {
  constructor(client) {
    this.client = client;
  }

  async run(reaction, user) {
    const message = reaction.message;
    if (reaction.emoji.name !== '⭐') return;
    const { starboardChannel } = this.client.settings.get(message.guild.id);
    const fetch = await message.guild.channels.find('name', starboardChannel).fetchMessages({ limit: 100 });
    const stars = fetch.find(m => m.embeds[0].footer.text.startsWith('⭐') && m.embeds[0].footer.text.endsWith(reaction.message.id));
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
      const embed = new RichEmbed()

      const starMsg = await message.guild.channels.find('name', starboardChannel).fetchMessage(stars.id);
      await starMsg.edit({ embed });
    }
  }

  extension(reaction, attachment) {
    const imageLink = attachment.split('.');
    const typeOfImage = imageLink[imageLink.length - 1];
    const image = /(jpg|jpeg|png|gif)/gi.test(typeOfImage);
    if (!image) return '';
    return attachment;
  };
};
```