# ternary operators

## What is a ternary operator?

A ternary operator is some operation operating on 3 inputs. It's oftenly used as a shortcut for an if/else statement, and is also known as a conditional operator.

## How to use ternary operators

`expression ? if_true : if_false`

One can use this in multiple ways as it means you don't need to use loads of if/else statements.

An example would be for a command that can take a mention as an argument or no mention at all.

```js
const user = message.mentions.users.first();
if(user) {
    message.channel.send(user.displayAvatarURL());
}

if(!user) {
    message.channel.send(message.author.displayAvatarURL());
}
```

 It can be rather tedious to use a lot of if/else for simple conditions as it bloats the code. A simplified version using ternary operators would be:
```js
const user = message.mentions.users.size ? message.mentions.users.first() : message.author;
message.channel.send(user.displayAvatarURL());
```

Or for a more simplified version:

```js
message.channel.send((message.mentions.users.size ? message.mentions.users.first() : message.author).displayAvatarURL());

```

What does this code example do?

* `message.mentions.users.size` This will check if in the message, there is a user mention, 

* `message.mentions.users.first()` This will the set the variable `user` as the first user mention if the original condition is false.

* `message.author` This then sets the variable `user` as the message author [User instance](https://discord.js.org/#/docs/main/master/class/User).

* `displayAvatarURL()` This is a method to obtain the URL of a user's avatar.

In pseudocode:

The user constant is defined as what the ternary condition returns: if the message mentioned at least one user, define it as the first user mentioned, otherwise set it as the author of the message.

You can see how it cut down so many lines of code and makes it look neater.

## What does it actually do?

* The first operand is implicitly converted to bool. It is evaluated and all side effects are completed before continuing.

* If the first operand evaluates to true (1), the second operand is evaluated.

* If the first operand evaluates to false (0), the third operand is evaluated.

 ## More examples using ternary operators

Not every user has a nickname, so rather than it returning `undefined`, you have have it return `None` or their actually username.

```js
const nickname = message.member.nickname ? message.member.nickname : 'This user does not have a nickname.';
message.channel.send(nickname);
 ```

Sometimes users are offline, so when you try to obtain their activity, the bot may crash, there is a simple solution for this.

 ```js
const game = message.author.presence.activity ? message.author.presence.activity.name : 'None';
message.channel.send(game);
 ```

Here's an example for the message author, but we can also use it for mentions, again, using ternery operators:

```js
const user = message.mentions.users.size ? message.mentions.users.first() : message.author;
const game = user.presence.activity ? user.presence.activity.name : 'None.';
message.channel.send(game);
```

This example can be used to get the game activity of **any** user in the guild.
