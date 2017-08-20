# Command with arguments

In [Your First Bot](/getting-started/your_basic_bot.md), we explored how to make more than one command. These commands all started with a prefix, but didn't have any _arguments_ : extra parameters used to vary what the command actually does.

## Creating an array of arguments

The first thing that we need to do to use arguments, is to actually separate them. A command with arguments would normally look something like this:  
`!mycommand arg1 arg2 arg3`

In [Your First Bot](/getting-started/your_basic_bot.md) we actually simplify our task just a bit: our check for commands uses `startsWith()`:

```js
if (message.content.startsWith(config.prefix + "ping")) {
  message.channel.send("pong!");
}
```

This means that `!ping whaddup` or `!ping I'm a little teapot` would both trigger the command, and the bot would respond "pong!". It would just ignore everything after the command because of course we're not doing anything with it.

Let's start with creating an _Array_ containing each word after our command, using the `.split()` function... with a `regex`:

```js
if (message.content.startsWith(config.prefix + "ping")) {
  const args = message.content.split(/\s+/g);
}
```

> You *could* just split using a space `split(" ")` ... however doing this would break if you add an extra space. Especially an issue with mentions on mobile which sometimes add a new space before. Regex is fast enough for the purpose, and this will split on "any number of spaces between each word" instead of "once for each space".

This generates an array that would look like `["!ping", "I'm", "a", "little", "teapot"]`, for instance. We can then access any part of that array using `args[0]` to `args[4]`, which return the string in that array position. And if you want to be more precise and don't want `args` to contain the `command`, you can just remove it from the array: `const args = message.content.split(/\s+/g).slice(1);`.

> For more information on arrays, see [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

If you want, you can then specify argument _names_ by referring to the array positions:

```js
if (message.content.startsWith(config.prefix + "asl")) {
  const args = message.content.split(/\s+/g).slice(1);
  let age = args[0]; // yes, start at 0, not 1.
  let sex = args[1];
  let location = args[2];
  message.reply(`Hello ${message.author.name}, I see you're a ${age} year old ${sex} from ${location}. Wanna date?`);
}
```

And if you want to be **really** fancy with ECMAScript 6, here's an awesome one \(requires Node 6!\):

```js
if (message.content.startsWith(prefix + "asl")) {
  let [age, sex, location] = message.content.split(/\s+/g).slice(1);
  message.reply(`Hello ${message.author.username}, I see you're a ${age} year old ${sex} from ${location}. Wanna date?`);
}
```

> This is called [Destructuring](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) and it's awesome!

## Grabbing Mentions

Another way to use arguments, when the command should target a specific user \(or users\), is to use _Mentions_. For instance, to kick annoying shitposters with `!kick @Xx_SniperBitch_xX @UselessIdiot` can be done with ease, instead of attempting to grab their ID or their name.

In the context of the `message` event handler, all mentions in a message are part of the `msg.mentions` array. Each value in the array is a full `user` resolvable so you can get their ID, name, etc.

Let's build a quick and dirty `kick` command, then. No error handling or mod checks - just straight up! (*Cul Sec*, as the French would say):

```js
// Kick a single user in the mention
if (message.content.startsWith(config.prefix + "kick")) {
  let member = message.mentions.members.first();
  member.kick();
}
```

This would be called with, for example, `!kick @AnnoyingUser23`

## Variable Length arguments

Let's make the above kick command a little better. Because Discord now supports kick *reasons* in the Audit Logs, the Discord.js `kick()` command also supports an optional `reason` argument. But, because the reason can have multiple words in it, we need to *join* all these words together.

So let's do this now, with what we've already learned, and a little extra:

```js
if(message.content.startsWith(config.prefix + "kick")) {
  let member = message.mentions.members.first();
  let reason = message.content.split(/\s+/g).slice(2).join(" ");
  member.kick(reason);
}
```

So, the reason is obtained by:
- Grabbing the message content
- Splitting it by spaces
- Removing the first 2 elements (the command, `!kick` and the mention, which looks like `<@1234567489213>`
- Re-joining the rest of the array elements with a space.

To use this command, a user would do something like: `!kick @SuperGamerDude Obvious Troll, shitposting`.

> If you're thinking, "What if I have more than one argument with spaces?", yes that's a tougher problem. Ideally, if you need more than one argument with spaces in it, do not use spaces to split the arguments. For example, `!newtag First Var Second Var Third Var` won't work. But `!newtag First Var;Second Var;Third Var;` could work by removing the command, splitting by `;` then splitting by space. Not for the faint of heart!

### Let's be fancy with ES6 again!

Destructuring has a `...rest` feature that lets you take "the rest of the array" and put it in a single variable. To demonstrate this, let me show you part of a code I use in a "save message" command. Basically, I store a message to a database, with a name. I call this command using: `!quote <channelid> <messageID> quotename note`, where `quotename` is a single word and `note` may be multiple words. The *actual* command code is unimportant. However, the way I process these arguments, is useful:

```js
if(message.content.startsWith(config.prefix + "quote")) {
  const [channelid, messageid, quotename, ...note] = message.content.split(/\s+/g).splice(1);
  // I also support "here" as a channelID using this:
  const channel = channelid == "here" ? message.channel : client.channels.get(channelid);
  // I do the same with message ID, which can be "last":
  const message = messageid === "last" ? msg.channel.messages.last(2)[0] : await channel.messages.get(messageid);
  // pretend for a second this is the rest of the function:
  insertInDB(quotename, channel.id, message.id, note.join(" "));
}
```

A few notes on this code, because I will admit there's some new concepts in it you might not know:

- `...note` is "the rest of the array arguments", so it would be `["this", "is", "a", "note"]`, that's why we .join(" ") when we use it.
- `const myVar = condition ? codeWhenConditionTrue : codeWhenFalse;` is called the "ternary operator" in javascript and makes some conditions much more simpler.
- `<Collection>.last(2)[0]` is only available on discord.js#master (the 12.0 beta version) which is not out at the time of writing. The alternative is complex, and beyond what this page tries to teach, so just know it gets "the second to last message".

## Going one step further

Now, there's most definitely always room for some optimization, and better code. At this point, "parsing arguments" becomes something you might realize is necessary for *all* of your commands, and writing ".startsWith()" for every command is dull and boring. So, as your next step, consider looking at making [A Basic Command Handler](/coding-guides/a-basic-command-handler.md). This **greatly** simplifies the creation of new commands.
