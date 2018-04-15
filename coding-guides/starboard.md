# Starboard

This is a long awaited feature, requested by many people.

Let's begin by talking about what a starboard is. This is an example taken from the Discord.js Official Server.

![Starboard](/assets/starboard.png)

A starboard is a popular feature in bots that serve as a channel of messages that users of the server find funny, stupid, or both! To make a functioning starboard, we need to monitor for a reaction being added to a message, and we'll do this with the `messageReactionAdd` and `messageReactionRemove` events.

> Before we start, there is one thing you need to know. This code is compatible with only Guidebot Class. This is due to a restriction with passing `client` to the messageReactionAdd and messageReactionRemove events.

So, let's begin!

In this block, we just do some simple setup for later on. For ease, I personally define the message object as `message`, but this is completely optional. Next, we grab the starboardChannel key from the guilds settings. 

Then, we preform a couple of checks on the reaction and the message. First, we check if the reaction is **NOT** the unicode star emote. Next, we preform two checks on the message, checking to see if the user who added the reaction authored the message, if the user who sent the message is the person who reacted to it, and if the message author is a bot. If none of these checks return true, we're good to move on.

```js
module.exports = class {
  constructor(client) {
    this.client = client;
  }
  
  // This is where all the action happens. 
  async run(reaction, user) {
    const message = reaction.message;
    if (reaction.emoji.name !== '⭐') return; // This is the first check where we check to see if the reaction is not the unicode star emote.
    if (message.author.id === user.id) return message.channel.send(`${user}, you cannot star your own messages.`); // Here we check to see if the person who reacted is the person who reacted is the person who sent the message.
    if (message.author.bot) return message.channel.send(`${user}, you cannot star bot messages.`); // This is our final check, checking to see if message was sent by a bot.
    const { starboardChannel } = this.client.settings.get(message.guild.id); // Here we get the starboard channel from the guilds settings.
    if (!message.guild.channels.exists('name', starboardChannel)) return message.channel.send(`It appears that you do not have a \`${starboardChannel}\` channel.`); // If there's no starboard channel, we stop the event from running any further, and tell them that they don't have a starboard channel.
  }
}
```

Now, this next block may look real complex, but it really isn't. Let's break it down.

First, we declare 6 variables here, fetch, stars, star, foundStar, image, and embed. `fetch` will fetch the last 100 messages in the starboard channel, `stars` attempts to find an embed who's footer ends with the message's id.

Next, we'll start an if statement for if there is already an embed in the starboard channel for this message, and we also define our final 4 variables. `star` is a simple regex to check how many stars the embed already has, `foundStar` allows us to use the color of the pre-existing embed, `image` checks if there is anything attached to the message, and `embed` is our new embed!

We also use a function that hasn't been talked about yet, and that is `this.extension`. It checks the message to see if it has any images. You'll see that later, as it's not _too_ important.

I told you it wasn't that complicated. Let's keep going.

```js
const fetch = await message.guild.channels.find('name', starboardChannel).fetchMessages({ limit: 100 }); // Here we fetch 100 messages from the starboard channel.
const stars = fetch.find(m => m.embeds[0].footer.text.startsWith('⭐') && m.embeds[0].footer.text.endsWith(message.id)); // We check the messages within the fetch object to see if the message that was reacted to is already a message in the starboard,
if (stars) { // Now we setup an if statement for if the message is found within the starboard.
  const star = /^\⭐\s([0-9]{1,3})\s\|\s([0-9]{17,20})/.exec(stars.embeds[0].footer.text); // Regex to check how many stars the embed has.
  const foundStar = stars.embeds[0]; // A variable that allows us to use the color of the pre-existing embed.
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : ''; // We use the this.extension function to see if there is anything attached to the message.
  const embed = new RichEmbed()
    .setColor(foundStar.color)
    .setDescription(foundStar.description)
    .setAuthor(message.author.name, message.author.displayAvatarURL)
    .setTimestamp()
    .setFooter(`⭐ ${parseInt(star[1])+1} | ${message.id}`)
    .setImage(image);
  const starMsg = await message.guild.channels.find('name', starboard).fetchMessage(stars.id); // We fetch the ID of the message already on the starboard.
  await starMsg.edit({ embed }); // And now we edit the message with the new embed!
}
```

Now, if you were to just use the code above, your starboard would only function if there was already a message in the starboard channel. Let's take care of that.

Here we add an if statement that mimics and is placed after the previous block, but this time manually setting the color of the embed, and also manually setting the amount of stars the embed will have.

```js
if (!stars) { // Now we use an if statement for if a message isn't found in the starboard for the message. 
  if (!message.guild.channels.exists('name', starboardChannel)) return message.channel.send(`It appears that you do not have a \`${starboardChannel}\` channel.`); // Once again, if there's no starboard channel, we stop the event from running any further, and tell them that they don't have a starboard channel.
  const image = message.attachments.size > 0 ? await this.extension(reaction, message.attachments.array()[0].url) : ''; // We use the this.extension function to see if there is anything attached to the message.
  if (image === '' && message.cleanContent.length < 1) return message.channel.send(`${user}, you cannot star an empty message.`); // If the message is empty, we don't allow the user to star the message.
  const embed = new RichEmbed()
    .setColor(15844367) // We set the color to a nice yellow here.
    .setDescription(message.cleanContent) // Here we use cleanContent, which replaces all mentions in the message with their equivalent text. For example, an @everyone ping will just display as @everyone, without tagging you!
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

  // Here we add the this.extension function to check if there's anything attached to the message.
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

Here, we very closely mimic the messageReactionAdd event, only this time we'll **subtract** 1 from the total amount of stars that are on the embed.'

There's only a slight difference between the `messageReactionAdd` and the `messageReactionRemove` event, and that is the `messageReactionAdd` event fires when a reaction is added, and the `messageReactionRemove` event fires when a reaction is removed.

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

Here's the finalized code block for the `messageReactionRemove` event.

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
      const starMsg = await message.guild.channels.find('name', starboardChannel).fetchMessage(stars.id);
      await starMsg.edit({ embed });
    }
  }

  // Now, it may seem weird that we use this in the messageReactionRemove event, but we still need to check if there's an image so that we can set it, if necessary.
  extension(reaction, attachment) {
    const imageLink = attachment.split('.');
    const typeOfImage = imageLink[imageLink.length - 1];
    const image = /(jpg|jpeg|png|gif)/gi.test(typeOfImage);
    if (!image) return '';
    return attachment;
  };
};
```