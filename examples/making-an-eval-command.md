# Making an Eval command

## What is eval exactly?

In JavaScript \(and node\), `eval()` is a function that evaluates any string _as javascript code_ and actually executes it. Meaning, if I `eval(2+2)` , eval will return `4`. If I eval `client.guilds.cache.size`, it'll return however many guilds the bot is currently on. And if I eval `client.guilds.cache.map(g=>g.name).join('\n')` then it will return a list of guild names separated by a line return. Cool, right?

## But eval is dangerous

I'll say this in a way that's probably dead simple to understand: _Giving someone access to_ `eval()` _is literally like sitting them **at your computer**, with **full admin access**, and then **stepping out of the room**._ `eval()` in browser javascript is trivial and not dangerous - you're running in your own browser, anything you fuck up is going to be on your own PC, not the web servers.

But `eval()` in **node** is really, really dangerous and powerful. Because it can run anything **you** run as a bot, and it can also run code you're not **expecting** to run, if someone else has access to it. **Node.js** has access to your **hard drive**. The whole thing. Every bit of it. To understand what this means, look at the following command: `rm -rf / --no-preserve-root` . Do you know what this command does? **It deletes your entire server's hard drives**. I mean, it only works on Linux, but most VPS systems and most hosting providers are on UNIX-based systems.

Another thing that's on your hard drive is passwords. Have a config file with your database password? I can get that. `config.json` with your token and other API keys in it? I can grab that easy. Hell I can just grab your token straight from the `client` object if I know how to.

## Securing your eval

So first, we need to understand the \#1 rule when using `eval` commands:

{% hint style="danger" %}
_**NEVER EVER GIVE EVAL PERMS TO ANYONE ELSE**_

I don't care if it's a server owner, someone you've been talking to for months, you **cannot** trust anyone with eval. There's only one exception to this rule: Someone you know **in real life** that you can punch in the face when they actually destroy half your server or mistakenly ban everyone in every server your bot is in. Eval bypasses any command-based permission you might have, it bypasses all security checks. Eval is all powerful.
{% endhint %}

So how do you secure it? Simple: only allow use from your own user ID. So for example my user ID is `139412744439988224` so I check whether the message author's ID is mine, which we added into our config at the start:

```javascript
{
  //the rest of the config
  "ownerID": "139412744439988224"
}
```

In the code for the bot:

```javascript
if (message.author.id !== config.ownerID) return;
```

It's as simple as that to protect the command directly inside of your condition or file or whatever. Of course, if you have some sort of command handler there's most likely a way to restrict to an ID too. This isn't specific to discord.js : there's always a way to do this. If there isn't \(if a command handler won't let you restrict by ID\), then you're using the **wrong lib**.

## Simple Eval Command

So now you've been thoroughly briefed on the dangers of Eval, let's take a look at how to implement a simple eval command.

First though I strongly suggest using the following function \(plop it outside of any event handler/functions you have, so it's accessible anywhere\). This function prevents the use of actual mentions within the return line by adding a zero-width character between the `@` and the first character of the mention - blocking the mention from happening.

{% hint style="info" %}
**EITHER** of the following clean snippets are _**REQUIRED**_ to make the eval work.
{% endhint %}

```javascript
const clean = async text => {
  if (typeof(text) === "string")
    return text
    .replace(/`/g, "`" + String.fromCharCode(8203))
    .replace(/@/g, "@" + String.fromCharCode(8203));
  else
      return text;
}
```

```javascript
const clean = async (text) => {
if (text && text.constructor.name == "Promise")
      text = await text;
    if (typeof text !== "string")
      text = require("util").inspect(text, { depth: 1 });

    text = text
      .replace(/`/g, "`" + String.fromCharCode(8203))
      .replace(/@/g, "@" + String.fromCharCode(8203));

    return text;
```

Alright, So let's get down to the brass tax: The actual eval command. Here it is in all its glory, assuming you've followed this guide all along:

```javascript
client.on("message", async (message) => {
  const args = message.content.split(" ").slice(1);
  if (message.content.startsWith(config.prefix + "eval")) {
    if (message.author.id !== config.ownerID) return;
      try {
        const evaled = eval(code);
        const cleaned = await clean(evaled);
        message.channel.send(`\`\`\`js\n${cleaned}\n\`\`\``);
      } catch (err) {
        message.channel.send(`\`ERROR\` \`\`\`xl\n${cleaned}\n\`\`\``);
      }
  }
});
```

That's it. That's the command. Note a couple of things though:

* Your IDE or editor might scream at the `eval(code)` for being `unsafe`. See the rest of this page for WHY it says that. Can't say you haven't been warned!
* If the response isn't a string, `util.inspect()` is used to 'stringify' the code in a safe way that won't error out on objects with circular references \(like most Collections\).
* If the response is more than 2000 characters this will return nothing.

{% hint style="info" %} This won't censor/remove your client token, to censor your token just add the following code.
{% endhint %}

```javascript
// You will need to pass the client into the clean function like so.
const clean = async (client, text) => { 
  // The rest of the code
}

// Then you will ned to place this inside the clean function, before the return
// text line;
text = replaceAll(text, client.token, "Not for you.");

// And finally, add this function to the file.
function replaceAll(haystack, needle, replacement) {
  return haystack.split(needle).join(replacement)
}
```

If you are using Node v15, you do not need to include the [`replaceAll`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replaceAll) function, as it is native to Node v15 but a slight modification to the code will be required.

{% hint style="danger" %}
**I AM NOT RESPONSIBLE IF YOU FUCK UP, AND NEITHER ARE ANY OF THE DISCORD.JS USERS AND DEVELOPERS**
{% endhint %}

Hopefully the warnings were clear enough to help you understand the dangers... but the idea of eval is still attractive enough that you'll use it for yourself anyway!
