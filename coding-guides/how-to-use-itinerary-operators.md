# Itinerary Operators

## What is an Itinerary Operator?

A ternary operator is some operation operating on 3 inputs. It's a shortcut for an if/else statement, and is also known as a conditional operator.


## How to use Itinerary Operators

`expression ? expression : expression`

One can use this in multiple ways as it means you don't need to use loads of if/else statements.

An example would be for a command that can take a mention as an argument or no mention at all.

```js
const user = message.mentions.users.first();
if(user) {
    message.channel.send(message.author.displayAvatarURL());
};

if(!user) {
    message.channel.send(message.author.displayAvatarURL());
};
```

As you can see, it can be rather tedious and can use multiple lines of unnecessary code. A simplified version using Itinerary Operators would be:
```js
const user = message.mentions.users.first() === undefined ? message.author : message.mentions.users.first();
message.channel.send(user.displayAvatarURL());
```

What this does in this current code example:

* `message.mentions.users.first() === undefined` This will check if in the message, there is not a user mention, 

* `message.author` This then sets the variable `user` as the message author (user object),

* `message.mentions.users.first()` This will the set the variable `user` as the first user mention if the original condition is false.


In sudo code:

if there is no user mentions, set the user variable as the message.author, if there is a mention, set the user variable as the user mention.

You can see how it cut down so many lines of code and makes it look neater.


## What does it actually do?

* The first operand is implicitly converted to bool. It is evaluated and all side effects are completed before continuing.

* If the first operand evaluates to true (1), the second operand is evaluated.

* If the first operand evaluates to false (0), the third operand is evaluated.


 ## More examples using Itinerary Operators

Not every user has a nickname, so rather than it returning `undefined`, you have have it return `None` or their actually username.

```js
const nickname = message.member.nickname === undefined ? 'None' : message.member.nickname;
message.channel.send(nickname)
 ```

Sometimes users are offline, so when you try to obtain their activity, the bot may crash, a simple solution to this is... Itinerary Operators!

 ```js
const game = message.author.presence.activity === null ? 'None' : message.author.presence.activity.name
message.channel.send(game)
 ```

Here's an example for the message author, but we can also use it for mentions, again, using Itinerary Operators:

```js
const user = message.mentions.users.first() === undefined ? message.author : message.mentions.users.first();
const game = user.presence.activity === null ? 'None' : user.presence.activity.name
```

This example can be used to get the game activity of **any** user in the guild.
