# Command with arguments

In [Your First Bot](../coding-walkthroughs/your_basic_bot.md), we explored how to make more than one command. These commands all started with a prefix, but didn't have any _arguments_ : extra parameters used to vary what the command actually does.

## Creating an array of arguments

The first thing that we need to do to use arguments, is to actually separate them. A command with arguments would normally look something like this:  
`!mycommand arg1 arg2 arg3`

In [Your First Bot](../coding-walkthroughs/your_basic_bot.md) we actually simplify our task just a bit: our check for commands uses `startsWith()`:

```js
if (<Message>.content.startsWith(prefix + "ping")) {
  <Message>.channel.sendMessage("pong!");
}
```

This means that `!ping whaddup` or `!ping I'm a little teapot` would both trigger the command, and the bot would respond "pong!". It would just ignore everything after the command because of course we're not doing anything with it.

Let's start with creating an _Array_ containing each word after our command, using the `.split()` function:

```js
if (<Message>.content.startsWith(prefix + "ping")) {
  let args = <Message>.content.split(" ");
}
```

This generates an array that would look like `["!ping", "I'm", "a", "little", "teapot"]`, for instance. We can then access any part of that array using `args[0]` to `args[4]`, which return the string in that array position. And if you want to be more precise and don't want `args` to contain the `command`, you can just remove it from the array: `let args = <Message>.content.split(" ").slice(1);`.

> For more information on arrays, see [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

If you want, you can then specify argument _names_ by referring to the array positions:

```js
if (<Message>.content.startsWith(prefix + "asl")) {
  let args = <Message>.content.split(" ").slice(1);
  let age = args[0]; // yes, start at 0, not 1. I hate that too.
  let sex = args[1];
  let location = args[2];
  <Message>.reply(`Hello ${<Message>.author.name}, I see you're a ${age} year old ${sex} from ${location}. Wanna date?`);
}
```

And if you want to be **really** fancy with ECMAScript 6, here's an awesome one \(requires Node 6!\):

```js
if (<Message>.content.startsWith(prefix + "asl")) {
  let [age, sex, location] = <Message>.content.split(" ").slice(1);
  <Message>.reply(`Hello ${<Message>.author.name}, I see you're a ${age} year old ${sex} from ${location}. Wanna date?`);
}
```

> This is called [Destructuring](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) and it's awesome!

## Grabbing Mentions

Another way to use arguments, when the command should target a specific user \(or users\), is to use _Mentions_. For instance, to kick annoying shitposters with `!kick @Xx_SniperBitch_xX @UselessIdiot` can be done with ease, instead of attempting to grab their ID or their name.

In the context of the `message` event handler, all mentions in a message are part of the `msg.mentions` array. Each value in the array is a full `user` resolvable so you can get their ID, name, etc.

Let's build a quick and dirty `kick` command, then.

```js
// Kick a single user in the mention
if (<Message>.content.startsWith(prefix + "kick")) {
  // I'll make a code example on how to check if the user is allowed, one day!
    let userToKick = <Message>.mentions.users.first();
    //we need to get a *GuildMember* object, mentions are only users. Then, we kick!
    <Message>.guild.member(userToKick).kick();
  // see I even catch the error!
}
```

If you want to support _multiple_ mentions, then you'll have to loop through each of the mentions. I mean, you _should_ know how to loop, but I'm feeling generous with spoonfeeding today:

```js
if (<Message>.content.startsWith(prefix + "kick")) {
  // I'll make a code example on how to check if the user is allowed, one day!
  <Message>.mentions.users.map( (user) => {
    userToKick.kick().catch(console.error);
  });
  // Yes that's right, I'm using ECMAScript 6 to confuse you again.
}
```

> There's a reason I'm adding error catching here. It's because the first time you use this command, you are most likely going to come to the realization that your bot does not have kick permissions. You're welcome.

## Variable Length arguments

Let's say you want to have a command that creates a new role. To be honest I wouldn't - roles should be created manually - but for the sake of argument let's do it.

We'll give the role command 3 arguments: A role _name_, a role _color_ and whether the role is _hoisted_ \(aka, shows separately in the user list\). The order of these arguments is important here: _hoisted_ and _color_ will always be one word, but the role _name_ can be variable. "Bot Commander" "Admin People", "Eternal Noobs" are all valid role names. So let's handle them right!

```js
if (<Message>.content.startsWith(prefix + "newrole")) {
  let args = <Message>.content.split(" ").slice(1);
  let color = args[0];
  let hoist = args[1] === "yes" || args[1] === "true" ? true : false; // google "ternary if javascript"
  let rolename = args.slice(2).join(" "); // remove first 2 args, then join array with a space
  guild.createRole()
  .then( newrole => {
    newrole.edit({hoist: hoist, name: rolename, color: color})
  }).catch(console.error);
}
```

> **Note**: Colors for roles are essentially _hex_ colors similar to html, but instead of \#FFFFFF for white, it's 0xFFFFFF. [Here's a handy reference color chart!](http://www.nthelp.com/colorcodes.htm)

To use this command, a user would do something like: `!newrole 0000FF yes Eternal Noob`.

> If you're thinking, "What if I have more than one argument with spaces?", yes that's a tougher problem. Ideally, if you need more than one argument with spaces in it, do not use spaces to split the arguments. For example, `!newtag First Var Second Var Third Var` won't work. But `!newtag First Var;Second Var;Third Var;` can work if you first remove the command \(with `var args = msg.content.split(" ")[1];`. Then you do `args = args.split(";");` and you get the arguments, properly separated!

### Let's be fancy with ES6 again!

Destructuring has a `...rest` feature that lets you take "the rest of the array" and put it in a single variable. Similarly to the above shortened version, you could thus do:

```js
if (<Message>.content.startsWith(prefix + "newrole")) {
  let [color, hoist, ...rolename] = <Message>.content.split(" ").slice(1);
  hoist = hoist === "yes" || hoist === "true" ? true : false; // Still gotta do this check!
  guild.createRole()
  .then( newrole => {
    newrole.edit({hoist: hoist, name: rolename, color: color})
  }).catch(console.error);
}
```



