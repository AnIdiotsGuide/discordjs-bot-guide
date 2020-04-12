# Command with arguments

## Creating an array of arguments

The first thing that we need to do to use arguments, is to actually separate them. A command with arguments would normally look something like this: `!mycommand arg1 arg2 arg3`

In this, we need to do 3 things:

* Remove the prefix
* Grab the _command_ part \(`mycommand`\)
* Grab the _array_ of _arguments_ which will be:

`['arg1', 'arg2', 'arg3']`

In the greatest majority of the code I've seen, arguments are _split_ at the beginning of the code, and each command will put the argument array back together as necessary within the command code.

In my experience, the best \(and most efficient\) way of separating all these things is the following 2 lines of code:

```javascript
  const args = message.content.slice(prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();
```

Let's break this down into what it _actually_ does, line by line.

* `.slice(prefix.length)` removes the prefix such as `!` or `+` from the message content, leaving `mycommand arg1 arg2 arg3`.
* `.trim()` ensures there's no extra spaces before/after the text.
* `.split(/ +/g)` splits the string by _one or many spaces_. Why not just by space? Because sometimes especially on mobile, you might have an extra space before or after mentions, or just straight up to an extra space by mistake. This means that `mycommand    arg1  arg2        arg3` will work just as well as if they only had 1 space.

On the second line:

* `args.shift()` where `shift()` will **remove one element from the array** and return it. This gives us `mycommand` that's returned, and the `args` array becomes only `['arg1', 'arg2','arg3']`
* `.toLowerCase()` so our command is always in lowercase, meaning `!Ping`, `!ping` and `!PiNg` will all work.

## Using the `command` variable properly

So now that we have our `command` variable, we no longer need to use the `if(message.content.startsWith(prefix+'command'))` for every command. We can simplify this by looking only at the `command` variable itself. For example, these 2 very basic commands:

```javascript
if(command === 'ping') {
  message.channel.send('Pong!');
} else
if (command === 'blah') {
  message.channel.send('Meh.');
}
```

Now isn't that just so pretty and clean? I love it!

Alternatively, you can also use this in a switch/case command block \(I don't like it, but to each their own, right?\):

```javascript
switch (command) {
  case "ping" :
    message.channel.send('Pong!');
    break;
  case "blah" :
    message.channel.send('Meh.');
    break;
}
```

Here's a complete example that's very often the command handler I've used as a base to build on:

```javascript
client.on("message", message => {
  if (message.author.bot) return;
  // This is where we'll put our code.
  if (message.content.indexOf(config.prefix) !== 0) return;
 
  const args = message.content.slice(config.prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();
 
  if(command === 'ping') {
    message.channel.send('Pong!');
  } else
  if (command === 'blah') {
    message.channel.send('Meh.');
  }
});
```

## Working with the arguments

Alright let's get to the meat of this page: actually using the `args` array in a few command examples.

The first one is a perfectly useless command, but for the life of me, I can't actually think of a really simple command using only one-word arguments. So here is a ridiculous ASL command:

```javascript
if (command === "asl") {
  let age = args[0]; // Remember arrays are 0-based!.
  let sex = args[1];
  let location = args[2];
  message.reply(`Hello ${message.author.username}, I see you're a ${age} year old ${sex} from ${location}. Wanna date?`);
}
```

And if you want to be **really** fancy with ECMAScript 6, here's an awesome one \(requires Node 6!\):

```javascript
if (command === "asl") {
  let [age, sex, location] = args;
  message.reply(`Hello ${message.author.username}, I see you're a ${age} year old ${sex} from ${location}. Wanna date?`);
}
```

{% hint style="info" %}
This is called [Destructuring](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) and it's awesome!
{% endhint %}

## Grabbing Mentions

Another way to use arguments, when the command should target a specific user \(or users\), is to use _Mentions_. For instance, to kick annoying shitposters with `!kick @Xx_SniperBitch_xX @UselessIdiot` can be done with ease, instead of attempting to grab their ID or their name.

In the context of the `message` event handler, all mentions in a message are part of the `msg.mentions` object. This object then contains multiple [Collections ](../understanding/collections.md)of different mention types. Here are the various available mention types: 

* `message.mentions.members` contains all @mention as [GuildMember ](https://discord.js.org/#/docs/main/stable/class/GuildMember)objects. 
* `message.mentions.users` contains all @mention as [User ](https://discord.js.org/#/docs/main/stable/class/User)objects. 
* `message.mentions.roles` contains all @role mention as [Role ](https://discord.js.org/#/docs/main/stable/class/Role)objects. 
* `message.mentions.channels` contains all \#channel mentions as [TextChannel](https://discord.js.org/#/docs/main/stable/class/TextChannel) objects.

Each of are collections so any collection method can be used on them. The most common method to use on mentions is .first\(\) which gets the very first mention, since there is often only one of them.

Let's build a quick and dirty `kick` command, then. No error handling or mod checks - just straight up! \(_Cul Sec_, as the French would say\):

```javascript
// Kick a single user in the mention
if (command === "kick") {
  let member = message.mentions.members.first();
  member.kick();
}
```

This would be called with, for example, `!kick @AnnoyingUser23`

## Variable Length arguments

Let's make the above kick command a little better. Because Discord now supports kick _reasons_ in the Audit Logs, the Discord.js `kick()` command also supports an optional `reason` argument. But, because the reason can have multiple words in it, we need to _join_ all these words together.

So let's do this now, with what we've already learned, and a little extra:

```javascript
if(command === "kick") {
  let member = message.mentions.members.first();
  let reason = args.slice(1).join(" ");
  member.kick(reason);
}
```

So, the reason is obtained by removing the first elements \(the mention, which looks like `<@1234567489213>`\) and re-joining the rest of the array elements with a space.

To use this command, a user would do something like: `!kick @SuperGamerDude Obvious Troll, shitposting`.

Here's another example, with a super simple command, the `say` command. It makes the bot say what you just sent, and then delete your message:

```javascript
if(command === "say"){
  let text = args.join(" ");
  message.delete();
  message.channel.send(text);
}
```

{% hint style="info" %}
If you're thinking, "What if I have more than one argument with spaces?", yes that's a tougher problem. Ideally, if you need more than one argument with spaces in it, do not use spaces to split the arguments. For example, `!newtag First Var Second Var Third Var` won't work. But `!newtag First Var;Second Var;Third Var;` could work by removing the command, splitting by `;` then splitting by space. Not for the faint of heart!
{% endhint %}

## Going one step further

Now, there's most definitely always room for some optimization, and better code. At this point, "parsing arguments" becomes something you might realize is necessary for _all_ of your commands, and writing "\(command === 'thing'\)" for every command is dull and boring. So, as your next step, consider looking at making [A Basic Command Handler](a-basic-command-handler.md). This **greatly** simplifies the creation of new commands.

