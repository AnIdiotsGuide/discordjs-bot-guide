# Selfbots, the greatest thing in the universe

So, my friend, you are wondering what is a selfbot? Well let me tell you a little story.

The first time you see it, you wonder what happened: someone typed for 2 seconds, and a wall of text appears as if by magic. Or something appears and then all of that person's messages disappear one after the other, relegated to `/dev/null` or some other weird place.

That, my friend, is selfbots in action.

## What is a selfbot?

First, let's define what a _userbot is_. A Userbot is **a bot that is running under a user account, instead of a true bot account**. Userbots do not have the blue \[BOT\] tag next to them... and they are **not** accepted by Discord as a valid way to make bots. At least, not since they introduced bot accounts under their [Discord Applications](https://discordapp.com/developers/applications/me) page.

On the other hand, a **selfbot** runs under an **active** user account. In other words, **your** account. When it posts a message, it's like you did. It can edit and delete your messages, has the same permissions you have, is on the same servers. **It's you**.

> **I AM NOT RESPONSIBLE AND CANNOT BE HELD LIABLE IF YOU MESS UP WITH SELFBOTS. THIS INCLUDES BUT IS NOT LIMITED TO LOSING PRIVILEGES, GETTING KICKED OR BANNED FROM SERVERS, OR BEING BANNED**

## What are the rules for selfbots?

I mentioned that _userbots_ are not tolerated by Hammer & Chisel. Selfbots, however, are tolerated under a specific set of semi-offical rules under which they turn a blind eye:

* A selfbot **must not**, under any circumstance, respond to other user's messages. This means it should not respond to commands, should not autoreply to certain keywords, etc. **You** must be the only one that can control it.
* A selfbot **must not**, under any circumstance, do "invite scraping". This is the action of detecting server invites in chat, and automatically joining a server using that invite. That is akin to creating a virus, and it is not acceptable.
* As selfbots run under your account they are subject to the same Terms of Service. That is to say, they should not _spam_, _insult or demean others_, _post NSFW material_, _spread viruses or illegal material_, etc. They have to follow the same rules that you follow.

**IF**, and **only if** you accept the above rules of selfbots, then you may proceed.

## How do I make a selfbot?

The code that creates selfbots is essentially the same as regular bots, because... well it's the same library, right?

### Respond only to you

The first difference is your `message` handler. It should start with a line that prevents the bot from interacting with anyone else's messages \(see rule \#1 above\):

```js
<Client>.on("message", <Message> => {
  if(<Message>.author !== <Client>.user) return;
  // rest of the code for commands go here
});
```

This condition says: "if the author of the message is **not** the bot user, stop processing". This is generic code that works for everyone, and it fulfills our first condition so **don't forget it**.

### The Token

The second difference, is how the bot logs in. To log a regular bot, you grab the Bot Token from the Applications Page, then put it in the `<Client>.login()` function.

In the case of a selfbot, this token can be obtained from the Discord application, from the Console:

1. From either the web application, or the installed Discord app, use the **CTRL+SHIFT+I** keyboard shortcut.
2. This brings up the **Developer Tools**. Go to the **Application** tab
3. On the left, expand **Local Storage**, then click on the discordapp.com entry \(it should be the only one\).
4. Locate the entry called `token`, and copy it.

![](https://cdn.discordapp.com/attachments/222079895583457280/269164273152819200/user-token.gif)

**KEEP YOUR TOKEN SECRET, AND NEVER SHARE IT WITH ANYONE**

I've already covered why Bot Tokens should remain secret in the [Getting Started](../getting-started/the-long-version.md) guide. But this is **even more** critical. If someone gets a hold your personal token, **they are you**. They can pretend to be you, send messages as you. **They can also massively fuck up any server where you have permissions**. Like transfer server control to themselves. Have I stated clearly enough that you should be careful with your token? Ok, good!

## Example selfbot commands

Here are a few useful examples of commands that I like to use.

### Prune Command

A _Prune_ command is used to delete your own messages from the channel you're on. Since there is no way for any server owner to prevent you from deleting or editing your own messages, this will always work. The command is called as `/prune X` where 99 is the maximum number of messages to remove, and the prefix depends on your own prefix.

> `fetchMessages()` is limited to 100 messages total, and gets _all_ the messages and not just your own. This means you will probably never be able to delete 100 messages since they'll be mixed in with other people's. I generally don't use it to prune more than 10 messages anyway.

```js
<Client>.on("message", <Message> => {
  if (<Message>.author !== <Client>.user) return;
  var prefix = "/"; // always use a prefix it's good practice.
  if (!<Message>.content.startsWith(prefix)) return; // ignore messages that... you know the drill.
  // We covered this already, yay!
  const params = <Message>.content.split(" ").slice(1);
  if (<Message>.content.startsWith(prefix + "prune")) {
    // get number of messages to prune
    let messagecount = parseInt(params[0]);
    // get the channel logs
    <Message>.channel.fetchMessages({
        limit: 100
      })
      .then(messages => {
        let msg_array = messages.array();
        // filter the message to only your own
        msg_array = msg_array.filter(m => m.author.id === <Client>.user.id);
        // limit to the requested number + 1 for the command message
        msg_array.length = messagecount + 1;
        // Has to delete messages individually. Cannot use `deleteMessages()` on self<Client>s.
        msg_array.map(m => m.delete().catch(console.error));
      });
  }
});
```

I've included the author check, prefix and params slice at the top for this first example - the other examples won't have that.

### Custom /lenny and friends

I love lenny, he's really a great friend to have. And his buddy /justright is also pretty cool! This command is... not really a command in the traditional sense of the word, because it's not built with the regular `if(msg.content.startsWith("something"))` we've seen before.

What it does is, check if the message is _only_ the prefix + any of the names in a map. The map itself can be placed outside of the `message` handler, or right before the commands are called:

```js
var shortcuts = new Map([
  ["lenny", "( ͡° ͜ʖ ͡°)"],
  ["shrug", "¯\\_(ツ)_/¯"],
  ["justright", "✋😩👌"],
  ["tableflip", "(╯°□°）╯︵ ┻━┻"],
  ["unflip", "┬──┬﻿ ノ( ゜-゜ノ)"]
]);
```

There's technically only 2 "new" commands here. The rest are there because I like to do /tableflip on my mobile phone, and this does it for me! And you can add your own, of course. Like a /doubleflip, for example.

The next step is to add the check at the beginning of your message handler. Well not quite: this code should be _right after_ your prefix check, before you split into parameters, for maximum code efficiency.

```js
<Client> .on("message", <Message> => {
  if (<Message>.author !== <Client>.user) return;
  var prefix = "/";
  if (!<Message>.content.startsWith(prefix)) return;

  // custom shortcut check
  var command_name = <Message>.content.slice(1); // removes the prefix, keeps the rest
  if (shortcuts.has(command_name)) {
    // setTimeout is used here because of a bug in message delays in Discord.
    // Otherwise the message would edit and then "seem" to un-edit itself... ¯\_(ツ)_/¯
    setTimeout(() => {<Message>.edit(shortcuts.get(command_name))}, 50);
    return;
  }

  // The Rest of your code
  // ...
});
```

### A complete implementation

The examples above are from my own personal selfbot. For your viewing pleasure I have uploaded it to github and will continue to update it as I use it and add more features.

[〈evie.codes〉's Selfbot](https://github.com/eslachance/djs-selfbot-v9) &gt; Have fun!

