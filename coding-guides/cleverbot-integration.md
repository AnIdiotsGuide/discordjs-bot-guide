# Cleverbot Integration

I've had this request since I started my Idiot's Guide, in fact it was one of the very first requests I had, but I had a feeling it would be a disappointing short episode, maybe a 5 minute long episode. But for a written guide it'd be perfect!

So to get started, let's grab the example from [getting started](../getting-started/getting-started-long-version.md) and shove it in a file.

```javascript
const Discord = require("discord.js");
const client = new Discord.Client();

client.on("ready", () => {
  console.log("I am ready!");
});

client.on("message", (message) => {
  if (message.content.startsWith("ping")) {
    message.channel.send("pong!");
  }
});

client.login("superSecretBotTokenHere");
```

Once you've got that, we should go check out `cleverbot-node` on [npmjs.com](https://www.npmjs.com/package/cleverbot-node) and grab their example code

```javascript
var Cleverbot = require("cleverbot-node");
cleverbot = new Cleverbot;
cleverbot.configure({botapi: "IAMKEY"});
cleverbot.write(cleverMessage, function (response) {
   console.log(response.output);
});
```

> **NOTE:** As you can see in the above example taken from the npm page, you now need an API key, which you can get [here](http://www.cleverbot.com/api/), ~~_they do offer a free tier, but that's 5,000 api calls per month \(at the time of writing\), and please note it says it's a free **trial**._~~
>
> **UPDATE:** They no longer offer a free trial.



Alright, we've got both parts we need, now before we continue we should get the module installed, just run `npm i cleverbot-node` with the `--save` flag if you have a `package.json` file \(and you should!\).

Installed? Good! Now, let's get to the final step... the code.

We have both our example codes, now we need to combine them for a working bot.

> **NOTE:** A lot of the naive developers would just shove the cleverbot example straight in their message event and wonder why it wasn't working. It would create a new instance of Cleverbot and would eventually cause a memory leak.

Right, we need to take the first two lines of the cleverbot example...

```javascript
var Cleverbot = require("cleverbot-node");
cleverbot = new Cleverbot;
```

...and put them with our discord definitions.

```javascript
const Discord = require("discord.js");
const Cleverbot = require("cleverbot-node");
const client = new Discord.Client();
const clbot = new Cleverbot;
clbot.configure({botapi: "IAMKEY"});
```

I renamed `cleverbot` to `clbot` to reduce any possible confusion between the variable names as JavaScript is case sensitive.

Then we take the rest of the code and place that inside our message event handler, but for this example I only want the bot to talk to me in DM's, so we"ll check the channel `type` with the following code, you can make it respond on mentions or even in channels \(I would honestly advise against that.\)

```javascript
if (message.channel.type === "dm") {
  // Cleverbot code goes here.
}
```

Your code should look something like this...

```javascript
const Discord = require("discord.js");
const Cleverbot = require("cleverbot-node");
const client = new Discord.Client();
const clbot = new Cleverbot;

client.on("message", message => {
  if (message.channel.type === "dm") {
    clbot.write(message.content, (response) => {
      message.channel.startTyping();
      setTimeout(() => {
        message.channel.send(response.output).catch(console.error);
        message.channel.stopTyping();
      }, Math.random() * (1 - 3) + 1 * 1000);
    });
  }
});

client.on("ready", () => {
  console.log("I am ready!");
});

client.login("superSecretBotTokenHere");
```

If everything is as above, then just send your bot a DM and watch the magic unfold!

![Success!](../.gitbook/assets/cleverbot.png)

