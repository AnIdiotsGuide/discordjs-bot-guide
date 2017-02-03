# You think you're so clever do you?

I've had this request since I started my Idiot's Guide, in fact it was one of the very first requests I had, but I had a feeling it would be a disappointing short episode, maybe a 5 minute long episode. But for a written guide it'd be perfect!

So to get started, let's grab the example from [getting started](/getting-started/the-long-version.md) and shove it in a file.

```js
var Discord = require("discord.js");
var client = new Discord.Client();

client.on("message", message => {
    if (message.content.startsWith("ping")) {
        message.channel.sendMessage("pong!");
    }
});

client.on('ready', () => {
  console.log('I am ready!');
});

client.login("yourcomplicatedBotTokenhere");
```

Once you've got that, we should go check out `cleverbot-node` on [npmjs.com](https://www.npmjs.com/package/cleverbot-node) and grab their example code

```js
var Cleverbot = require('cleverbot-node');
cleverbot = new Cleverbot;
Cleverbot.prepare(function(){
  cleverbot.write(cleverMessage, function (response) {
    alert(response.message);
  });
});
```

Alright, we've got both parts we need, now before we continue we should get the module installed, just run `npm i cleverbot-node` with the `--save` flag if you have a `package.json` file \(and you should!\).

Installed? Good! Now, let's get to the final step... the code.

We have both our example codes, now we need to combine them for a working bot.

> NOTE: A lot of the naive developers would just shove the cleverbot example straight in their message event and wonder why it wasn't working. It would create a new instance of Cleverbot and would eventually cause a memory leak.

Right, we need to take the first two lines of the cleverbot example...

```js
var Cleverbot = require('cleverbot-node');
cleverbot = new Cleverbot;
```

...and put them with our discord definitions.

```js
const Discord = require("discord.js");
const Cleverbot = require('cleverbot-node');
const client = new Discord.Client();
const clbot = new Cleverbot;
```

As you can see, I changed a few things, we don't and shouldn't be hoisting them up with `var` and we don't plan on redefining them, so we'll use `const`. I also renamed `cleverbot` to `clbot` to reduce any possible confusion.

Then we take the rest of the code and place that inside our message event handler, but for this example I only want the bot to talk to me in DM's, so we'll check the channel `type` with the following code

```js
if (message.channel.type === 'dm') {
  // Cleverbot code goes here.
}
```

Your code should look something like this...

```js
const Discord = require("discord.js");
const Cleverbot = require('cleverbot-node');
const client = new Discord.Client();
const clbot = new Cleverbot;

client.on("message", message => {
  if (message.channel.type === 'dm') {
    let args = message.content.split(' ').slice(1);
    CleverBot.prepare(() => {
      clbot.write(args, (response) => {
        message.channel.startTyping();
        setTimeout(() => {
          message.channel.sendMessage(response.message).catch(console.error);
          message.channel.stopTyping();
        }, Math.random() * (1 - 3) + 1 * 1000);
      });
    });
  }
});

client.on('ready', () => {
  console.log('I am ready!');
});

client.login("yourcomplicatedBotTokenhere");
```

If everything is as above, then just send your bot a DM and watch the magic unfold!  
![Success!](/assets/cleverbot.png)

