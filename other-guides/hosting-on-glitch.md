# Hosting on Glitch.com

{% hint style="danger" %}
It is strongly advised you actually shell out a couple dollars a month and get an actual real VPS, due to the free nature of Glitch it has led to a high number of Glitch based IP's getting API banned by Discord due to abuse and now Glitch are actively banning users for using external site pingers such as Uptimerobot.
{% endhint %}

{% embed url="https://www.youtube.com/watch?v=xX4AcRFojk4" %}

_**HOLY FUCKING HELL GUYS I'VE FOUND IT**_

I've finally found it. I found a free hosting that you guys can enjoy and put your bots on, that's not like the most complicated 4D puzzle in the world to get functional, and that _actually_ supports not only the most recent version of node, but also saving files locally so you can have a simple bot with persistence without so much hassle you'd be better off waiting until you had a job to get hosting.

So OH MY GOD, let's get to it right now because I'm so excited. I mean there's _tiny_ little things you have to be weary of, but it's, like, **Almost Paradise**. Let's knock on heaven's door.

## Getting Code Running

Ok like, have you ever seriously looked into getting a bot up on heroku? God it's a mess of setups and configurations and _holy crap what the hell is a worker and a procfile?????_

Glitch.com has **none** of this. Getting code up and running is as simple as:

* Open your browser to [Glitch.com](https://glitch.com/)
* **Completely ignore the childish drawings of a 3 year old on the front page** \(trust me on this one, don't look\)
* Click on **Start a New Project**, then **Create a Node App**
* You're... well technically you're done.

That's right. You can start coding right away, technically. The default project is an express.js website. **Don't delete that yet**, we'll still need some of that code later on. Because, as simple as this might be and, well, really is actually, there's still a few things that we need to set up, including making sure the project stays online, and securing it against prying eyes.

## Create your account

In order to not ever lose access to your code, the first thing you need to do is to create an account.

* Click **Sign In** at the top-right of the page
* Choose either Github or Facebook to login.
* Yeah ok the project is now yours.

Toldja this shit was simple.

## Configure the Project

So here's a few things about the project that we need to configure, for a few reasons.

First off, **Secure the project**. By default, anyone with your project's name can access your code directly. They can't _edit_ it but they can snoop in and look at your code. And, btw, I haven't found a way to un-share someone that's viewed your project yet \(I'm talking to Glitch to get that fixed\).

* Click on the **Share** Button at the top of the file list, besides your name.
* Click on **Make Private**.
* You can still invite people to view and collaborate later, with the link provided.

{% hint style="warning" %}
If you make your project private, people can snoop if you accidentally give out the project name and see the tokens in your env file, if you leave it public they can view your code, but cannot view any tokens inside the env file.
{% endhint %}

The next thing is, **Name the project**. Now, projects work through express.js whether you really want to or not. Later on you can learn to make a dashboard but for now, we just need to set it up to keep it alive.

* Click on the project name at the top-left of the screen \(mine was `best-glue`, these guys know what I sniff I tell ya!\)
* In the pop-up click on the name at the top \(_slightly_ counter-intuitive, but yeah that's how you rename it\)
* The name you choose \(it must be unique and not taken by anyone else\) will be your "site's" subdomain address.
* While you're at it, give it a description if you really want to.

Finally, we need to **Disable some auto-save features**. Glitch automatically saves the file, quite literally, on every keypress you make. And restarts it. This is not only slightly visually annoying, but also damaging to bots - the Discord API will reset your bot's token if you login 1000 times in a day. That means, if you type 1000 characters in your code, as it is. QUITE an issue.

* Create a new file in the project, and call it `watch.json`
* Paste in the following code in it:

```javascript
{
  "install": {
    "include": [
      "^package\\.json$",
      "^\\.env$"
    ]
  },
  "restart": {
    "exclude": [
      "^public/",
      "^dist/"
    ],
    "include": [
      "\\.js$",
      "\\.json"
    ]
  },
  "throttle": 900000
}
```

This number, `900000`, means that every 15 minutes, if any files have changed, the bot will restart. Now there is a caveat here, which is that this also means any change you do in the bot will not take effect \(will not reboot\) until, up to, 15 minutes. But hey. It's free, let's not look a gift horse in the mouth!

Oh one last thing for you crazy people with light-sensitive eyes \(aka dark theme users\) : click on your avatar at the top right, then click on "Change Theme". **I know, right? You're welcome!**

## Keeping the project "alive"

Alright so, like, Glitch is made to be a _web_ hosting really, and will "sleep" after 5 minutes if it receives no HTTP request. However, there is a very convenient way to keep it alive, which is actually provided by the app itself - the `express.js` module is pre-installed, and all you need to do is to "ping" it every 5 minutes to make sure it doesn't sleep. These lines of code in your project \(either the main file or any module you call on bootup\) should to the trick for now:

```javascript
const http = require('http');
const express = require('express');
const app = express();
app.get("/", (request, response) => {
  console.log(Date.now() + " Ping Received");
  response.sendStatus(200);
});
app.listen(process.env.PORT);
setInterval(() => {
  http.get(`http://${process.env.PROJECT_DOMAIN}.glitch.me/`);
}, 280000);
```

What does this do? Keeps an express.js server alive, which does not really affect your project in and of itself, and pings itself every 5 minutes, so it never shuts off. Awesome.

{% hint style="info" %}
For best results, have an outside source pinging the project address as well, glitch.com suggests using [Uptime Robot](https://uptimerobot.com/).
{% endhint %}

## Package.json

There are 2 things that you much change in the project's package.json file in order to ensure that your project will actually work.

First, you must provide for a _node.js version_ if your project requires a higher version of node \(for instance, 8.4.0\). This is done with the `engines` key, as such: `"engines": { "node": "8.4.0" }`

Second, you must provide for the `start` script. A lot of us just generally configure the `main: index.js` key and this is not sufficient. You must provide for a start script:

```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
```

I show the `test` script here because this is by default in any project where `npm init` was used, so it's a good point of reference. Here's a full package.json, this one is guidebot's modified version:

```javascript
{
  "name": "guidebot",
  "version": "2.0.3",
  "description": "A boilerplate example bot with command handler and reloadable commands. Updated and Maintained by the Idiot's Guide Community",
  "main": "node index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "engines": { "node": "8.4.0" },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/AnIdiotsGuide/guidebot.git"
  },
  "author": "The Idiot's Guide Community",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/AnIdiotsGuide/guidebot/issues"
  },
  "homepage": "https://github.com/AnIdiotsGuide/guidebot#readme",
  "dependencies": {
    "discord.js": "^11.2.1",
    "enmap": "^0.3.2",
    "moment": "^2.18.1",
    "moment-duration-format": "^1.3.0",
    "express": "^4.15.5"
  }
}
```

Another change is that your `config.json` file or `config.js` file is **insecure** if you share your project. The easiest way to fix this is to use environment variables. Open the `.env` file, and add the following line:

```text
TOKEN=MzUzOTUxODYwOTA3OTY2NDY0.DI3K3w.VN1Gvsl7CSh2IYIELJDJAFejH4w
```

{% hint style="info" %}
Obviously use your real token, duh!
{% endhint %}

You can then access this from anywhere using `process.env.TOKEN`, so again with the guidebot example, you would do the following in `config.js`:

```javascript
  // Your Bot's Token. Available on https://discordapp.com/developers/applications/me
  "token": process.env.TOKEN,
```

### F.A.Q.

* `X was compiled against a different Node.js version`

That error means you forgot to set your engines in your package file, make sure you have added `"engines": { "node": "8.4.0" }` to your package, you can also use the following commands in the console \(you can access the console by going to your Project Name &gt; Advanced Options &gt; Open Console\), `nvm use NODE VERSION`, followed by `npm rebuild`, that should eliminate the error.

* "My bot goes down after X"

Glitch puts projects to sleep after 5 minutes of inactivity, with the code you used near the start of the guide you should have at least one step towards keeping your bot online 24/7, how ever there is another tool you can use, and this is promoted by the Glitch team, and that tool is [Uptime Robot](https://uptimerobot.com/).

First you will need to go to the web address of your project \(`https://project-name.glitch.me`\). Then login to Uptime Robot and click on `+ Add new monitor`, set **monitor type** as `HTTP(S)`, give the monitor a name and in the **URL** field paste in your glitch project URL. Don't forget to set the **monitoring interval** to `Every 5 minutes (default)`, click save and job's done!

### Getting Help

Here are a few Glitch resources:

* [Glitch Discourse](https://support.glitch.com/) \(their Q&A/Forum place\)
* [Glitch FAQ](https://glitch.com/faq) \(some pertient technical details\)

