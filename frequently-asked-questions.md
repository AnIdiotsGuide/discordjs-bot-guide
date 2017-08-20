# Frequently Asked Questions

In this page, some very basic, frequently-asked questions are answered. It's important to understand that **these examples are generic** and will most likely not work if you just copy/paste them in your code. You need to **understand** these lines, not just blindly shove them in your code.

## Code Examples

## Bot and Bot Client

```js
// Set the bot's "Playing: " status (must be in an event!)
client.on("ready", () => {
    client.user.setGame("with my code");
});
```

```js
// Set the bot's online/idle/dnd/invisible status
client.on("ready", () => {
    client.user.setStatus("online");
});
```

## Users and Members

```js
// Get a User by ID
client.users.get("user id here");
// Returns <User>
```

```js
// Get a Member by ID
message.guild.members.get("user ID here");
// Returns <Member>
```

```js
// Get a Member from message Mention
message.mentions.members.first();
// Returns <Member>
```

```js
// Send a Direct Message to a user
message.author.send("hello");
```

```js
// Mention a user in a message
message.channel.send(`Hello ${user}, and welcome!`);
// or
message.channel.send("Hello " + message.author.toString() + ", and welcome!");
```

## Channels and Guilds

```js
// Get a Guild by ID
client.guilds.get("the guild id");
// Returns message.guild
```

```js
// Get a channel by ID
client.channels.get("the channel id");
// Returns message.channel
```

```js
// Get a Channel by Name (note: THIS IS NOT RECOMMENDED as more than one channel can have the same name!)
message.guild.channels.find("name", "channel-name");
// returns message.channel
```

## Common Errors & Fixes

### Cannot find module `discord.js`

#### Problem:

You didn't install Discord.js or installed it in the wrong folder

#### Solution:

* Make sure you are in the **correct** folder where you have your bot's files
* SHIFT+Right-Click in the folder and select **Open command window here**
* Run `npm init -y`, and hit enter until the wizard is complete
* Run `npm i -S discord.js` again to install Discord.

### Unexpected End of Input

#### Problem:

```
});
  ^
SyntaxError: Unexpected end of input
```

#### Solution:

Your code has an error somewhere. This is _impossible_ to troubleshoot without the **complete** code, since the error can be anywhere \(in fact the error stack often tells you it's at the end of your code\).

The following trick is a lifesaver, so pay attention: Your code editor is trying to help you. Whatever editor you're using \(except notepad++.exe. Don't use notepad++!\), clicking on any \(and I mean any\) special character such as parentheses, square brackets, curly braces, double and single quotes, will automatically highlight the one that matches it. The screenshot below shows this: I clicked on the curly brace at the bottom, it shows me the one on top by highlighting it. Learn this, and how different functions and event handlers "look" like.

![](assets/editorhelp.png)

You can check out [Installing and Using a Proper Editor](/other-guides/installing_and_using_a_proper_editor.md) to help in at least knowing there are errors _before_ running your bot code.
