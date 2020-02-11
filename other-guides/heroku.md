# Hosting on Heroku

Heroku is a free host that isn't quite meant for bots, but can serve as a host for small bots that are not too demanding on resources. In this guide we'll be taking a look at how to setup the Heroku CLI on your system, how to configure your project to be useable on Heroku, and how to push your changes to Heroku.

> Let's get 2 things out of the way first. First, Heroku can't handle Music Bots - don't even try, it won't be worth it. Second, you cannot _save data_ on a Heroku instance meaning you can't use sqlite, json data, enmap, or any other "save file on disk" concept. You'd need pgsql or a separate database instance.

There's a couple things you'll need before we get started.

* Node.js. Duh, because that's the whole point, right? 
* A Git Client. For Windows, use [GIT SCM](https://git-scm.com/). For other OSes, I'm sure you can figure it out, you googler you!
* A [Heroku](https://heroku.com/) account. 

## Installing Pre-Requisites

Let's assume for just a second that you're starting with a completely new bot. Let's see all the exact steps necessary, from an empty folder to a functional bot, one by one.

First we'll create a new empty folder. I'll call it `herokubot`. From this folder, you need a Command Prompt, PowerShell or Terminal window. If you don't know what I'm talking about, check out [getting started](../getting-started/getting-started-long-version.md). I'll call this the "prompt" from now on.

In the prompt, let's just prepare a couple things:

* Run `git init` to initialize this as a git repository. 
* Run `npm init -y` to automatically create a package.json. This is required because that's how heroku knows what to install for your project.
* Run `npm i discord.js dotenv` to install the simplest pre-requisites for a bot. For why we need `dotenv`, see [Using Environment Files](env-files.md)

## Setting up the project

There's 2 things that are required in the package.json specifically for Heroku \(and other hosts, really, but this isn't about those!\).

First, we need an `engines` property. This tells Heroku what version of Node to run your project on. The version here should be exactly equal to your local node installation for better compatibility.

In your package.json, add the following section \(changing node to the right version\):

```javascript
  "engines": {
    "node": "10.x",
    "npm": "*"
  },
```

Then, we actually need to tell Heroku how to start your project. This is done by adding the `start` entry into the `scripts` section. This section should already exist with the `test` entry, so we just need to add the new start entry. This should look like this:

```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
```

Save the file, you're done with it for the moment.

> Obviously, this assumes that your file is named `index.js`. If it's named something else, adjust to taste.

Next, you need a thing called a `Procfile`. This tells Heroku exactly how to run your project, specifically it says that it should _not_ expect an HTTP server to be started. If this isn't done, Heroku will stop your project after a couple minutes because it failed to bind an http server to the appropriate port.

Create a new file called `Procfile` in your project \(no extension, with the uppercase P\), and add in the following contents, which disables the need for a web server and lets your bot run:

```text
web: echo "I don't want a web process"
service: npm start
```

## Creating the main bot file

Next we need to actually create the bot. Per [Using Environment Files](https://github.com/AnIdiotsGuide/discordjs-bot-guide/tree/f386ebd2288dcc1f44537706bf463c5fd90b508e/other-guides/env-files.md), we're using `dotenv` to use environment variables. So here's a really basic bot file using this concept:

```javascript
// This line MUST be first, for discord.js to read the process envs!
require('dotenv').config(); 
const Discord = require("discord.js");
const client = new Discord.Client();

client.on("ready", () => {
  console.log("I am ready!");
});

client.on("message", message => {
  if (message.author.bot) return;
  // The process.env.PREFIX is your bot's prefix in this case.
  if (message.content.indexOf(process.env.PREFIX) !== 0) return;

  // This is the usual argument parsing we love to use.
  const args = message.content.slice(process.env.PREFIX.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  // And our 2 real basic commands!
  if(command === 'ping') {
    message.channel.send('Pong!');
  } else
  if (command === 'blah') {
    message.channel.send('Meh.');
  }
});

// There's zero need to put something here. Discord.js uses process.env.CLIENT_TOKEN if it's available,
// and this is what is being used here. If on discord.js v12, it's DISCORD_TOKEN
client.login();
```

Next, we create the `.env` file. This file is _only_ used locally, we won't be sending it to heroku or to github.

```text
CLIENT_TOKEN=MTg-this-IzNzU3OTA5NjA-is.not-DCeFB-a.real-r4DQlO-t0ken-qerT0
PREFIX=+
```

> Oh hey look, first, that's a fake token. Second, DO NOT add quotes around your token in this case!

Now, if you actually run the bot using `npm start`, it should print out "I am ready!". Hit CTRL+C to stop it for now!

## Using the Heroku CLI

The Heroku CLI essentially gives you an interactive prompt that you can use to interact with your Heroku service. Interestingly, Heroku recently added the CLI as an NPM module, making this part pretty trivial!

To install the CLI just go right ahead and run `npm i -g heroku`. It'll take a few seconds and then you're ready to roll out.

Next, you need to login. From your prompt, go ahead and run `heroku login`. This will ask you to enter your email, and password, for Heroku.

Then finally, you need to create an _application_ on Heroku. Go to your Dashboard, into [Create New App](https://dashboard.heroku.com/new-app). Enter a unique app name, like `my-super-original-bot` \(hereafter referred as &lt;appname&gt;\), in your closest region. 

One last thing before put in the last puzzle pieces, which is to configure the same Environment Variables that we have in our `.env` file. In the app's dashboard, go to the Settings tab, then click Reveal Config Vars. Add the 2 configuration variables. It should look like this:

![Heroku Config Vars](https://img.evie.codes/QNSpgc4)

Let's pre-emptively open up the logs so we can see what's going on on the server:

In your app's dashboard, click the "More" button at the top-right:

![Heroku Logs](https://img.evie.codes/hosTRYK)

It'll be empty for now, don't worry, let's keep going!

## Pushing the app

So we're on the last mile now! Everything you've done until now has prepared us for this moment.

* `git init` make our project git-enabled. 
* `heroku git:remote -a <appname>` to connect your local repository to the heroku app \(remember to put in your app name, no &lt;&gt; around it!\)
* `npm init -y` and the package.json file modifications prepared our project for the Heroku universe.

So let's finalize everything by saving our changes to `git` and pushing them to Heroku:

* Add all the files to git with `git add .` \(don't forget the dot!\)
* Prepare for pushing using commit: `git commit -m "Initial bot commit for Heroku"`
* Push the entire thing to Heroku: `git push heroku master`
* You'll see quite a lot of output logs which should end with something like "Verifying deploy... done."

OMG. That's... it. can you believe it? Your bot's now on Heroku and should really be running already!

Take a look at the logs on the dashboard, It should look like this \(Note: You'll have a few lines about processes crashing with Error Code 0, those are normal\):

![Heroku Success!](https://img.evie.codes/1Y6bJxZ)

And we're done! This should be enough to get you going, remember Heroku can't do a whole lot of processing, and you can't save files \(no file database, edited json, etc\). 

