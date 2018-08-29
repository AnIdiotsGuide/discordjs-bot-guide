---
description: >-
  Bots can access every emoji from all the guilds they're in, and the entire
  Unicode emojis. Learn how to use them in messages and in reactions!
---

# Using Emojis

Here's a fun fact you might not know about bots on Discord: They have access to every single "custom emoji" of every single guild they're in - for free. That's right, you have 1/4 of the features of Nitro, free in your bot, right now! In this page we'll be taking a look at how to take advantage of these emojis, how to access them and how to display them.

## What's an Emoji?

Let's start by tearing apart exactly what an Emoji is, how they're configured and how they're accessed. So here, we have an emoji:

![](https://cdn.discordapp.com/emojis/305818615712579584.png)

When I want to write this emoji in my chat, I simply type `:ayy:` and it turns into the above \(smaller, of course, but still\). But behind the scenes, 2 things happen for this emoji to show:

* Discord looks up the emoji in my list , finds the one with the name `ayy` and looks up its ID.
* It then sends the _actual_ emoji code to the server, which looks like this: `<:ayy:305818615712579584>`. This is the code that makes up the emoji.
* When a client receives the above, it looks up the URL for the Emoji from its ID, to get the image location. In this case, it's: `https://cdn.discordapp.com/emojis/305818615712579584.png`.
* As you can see the ID is the only thing that really matters in the URL. This ID is unique to each emoji.

## How does Discord.js store emojis?

There are two places where you can grab emojis using discord.js: in the client, and in the guilds. `client.emojis` is a collection of every emoji the client has access to, and `guild.emojis` is a collection of the emojis of a specific guild.

If you've learned anything from [Understanding Collections](using-emojis.md), you might already know how to get something by ID from a collection:

```javascript
const ayy = client.emojis.get("305818615712579584");
```

You might also know how to use `find` to get something with another property - so here, I can get `ayy` through its name:

```javascript
const ayy = client.emojis.find("name", "ayy");
```

## Outputting Emoji in chat

But how does one output that emoji to the chat? Well, just like users and roles, emojis have a special `.toString()` method that converts them to the appropriate format. So, `ayy.toString()` will actually output the `<:ayy:305818615712579584>` we saw above, which the client turns into a proper emoji.

You can also take advantage of concatenation and template literals to simplify the task, since they will automatically do the conversion for you:

```javascript
if(message.content === "ayy") {
   const ayy = client.emojis.find("name", "ayy");
   message.reply(`${ayy} LMAO`);
}
```

If you wanted to list all the emojis in a guild, a simple map operation on the collection should give you proper results:

```javascript
if (message.content === "listemojis") {
  const emojiList = message.guild.emojis.map(e=>e.toString()).join(" ");
  message.channel.send(emojiList);
}
```

In this example, you can list all custom emojis with \(emoji.id, emoji.image and emoji.name\).

```javascript
if (message.content === "listemojis") {
   const emojiList = message.guild.emojis.map((e, x) => (x + ' = ' + e) + ' | ' +e.name).join('\n');
   message.channel.send(emojiList);
}

example: 
450661466287112204 = :image: | name
```

## Reacting with Emojis

You can also use custom emojis as reactions to messages, using `message.react(emoji)`. In the case of custom emojis, you must use the emoji's ID, so you could do something like `message.react(ayy.id)` or `message.react("305818615712579584")` to add the `ayy` emoji as a reaction.

## But what about Unicode Emoji?

Don't forget there is a very extensive collection of emojis that are built into Discord that you can have access to. Discord uses Twemoji, provided by Twitter. You can use those emojis to react to messages directly.

The way that Discord expects those emojis however is that they have to be the _unicode_ character, not the "text". Meaning, you can't just do `message.send(":poop:")` and expect to see 💩 appear. You actually need to get the unicode value. How do you do that? Just escape the emoji in chat: `\:poop:` will show as 💩. You can copy/paste that inside your bot's code either in a message string, or as an emoji reaction such as `message.react("💩")`.

