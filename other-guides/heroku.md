# Hosting on Heroku

Heroku is a free host that isn't quite meant for bots, but can serve as a host for small bots that are not too demanding on resources. In this guide we'll be taking a look at how to setup the Heroku CLI on your system, how to configure your project to be useable on Heroku, and how to push your changes to Heroku.

> Let's get 2 things out of the way first. First, Heroku can't handle Music Bots - don't even try, it won't be worth it. Second, you cannot *save data* on a Heroku instance meaning you can't use sqlite, json data, enmap, or any other "save file on disk" concept. You'd need pgsql or a separate database instance.

There's a couple things you'll need before we get started. 

- NodeJS. Duh, because that's the whole point, right? 
- A Git Client. For Windows, use [GIT SCM](https://git-scm.com/). For other OSes, I'm sure you can figure it out, you googler you!
- A [Heroku](https://heroku.com/) account. 

## Installing Pre-Requisites

Let's assume for just a second that you're starting with a completely new bot. Let's see all the exact steps necessary, from an empty folder to a functional bot, one by one. 

First we'll create a new empty folder. I'll call it `herokubot`. From this folder, you need a Command Prompt, PowerShell or Terminal window. If you don't know what I'm talking about, check out [getting started](../getting-started/getting-started-long-version.md). I'll call this the "prompt" from now on. 

In the prompt, let's just prepare a couple things: 

- Run `git init` to initialize this as a git repository. 
- Run `npm init -y` to automatically create a package.json. This is required because that's how heroku knows what to install for your project.
- Run `npm i discord.js dotenv` to install the simplest pre-requisites for a bot. For why we need `dotenv`, see [Using Environment Files](./env-files.md)

## Setting up the project

There's 2 things that are required in the package.json specifically for Heroku (and other hosts, really, but this isn't about those!). 

First, we need an `engines` property. This tells Heroku what version of Node to run your project on. The version here should be exactly equal to your local node installation for better compatibility. 

In your package.json, add the following section (changing node to the right version): 

```json
  "engines": {
    "node": "8.x",
    "npm": "*"
  },
  ```

Then, we actually need to tell Heroku how to start your project. This is done by adding the `start` entry into the `scripts` section. This section should already exist with the `test` entry, so we just need to add the new start entry. This should look like this: 

```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
```

Save the file, you're done with it for the moment.

> Obviously, this assumes that your file is named `index.js`. If it's named something else, adjust to taste. 

Next, you need a thing called a `Procfile`. This tells Heroku exactly how to run your project, specifically it says that it should *not* expect an HTTP server to be started. If this isn't done, Heroku will stop your project after a couple minutes because it failed to bind an http server to the appropriate port. 

Create a new file called `Procfile` in your project (no extension, with the uppercase P), and add in the following contents: 

```
worker: npm start
```

## Creating the main bot file

Next we need to actually create the bot. Per [Using Environment Files](./env-files.md), we're using `dotenv` to use environment variables. So here's a really basic bot file using this concept: 

```js
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

Next, we create the `.env` file. This file is *only* used locally, we won't be sending it to heroku or to github. 

```
CLIENT_TOKEN=MTg-this-IzNzU3OTA5NjA-is.not-DCeFB-a.real-r4DQlO-t0ken-qerT0
PREFIX=+
```

> Oh hey look, first, that's a fake token. Second, DO NOT add quotes around your token in this case!

Now, if you actually run the bot using `npm start`, it should print out "I am ready!". Hit CTRL+C to stop it for now!

## Using the Heroku CLI

The Heroku CLI essentially gives you an interactive prompt that you can use to interact with your Heroku service. Interestingly, Heroku recently added the CLI as an NPM module, making this part pretty trivial! 

To install the CLI just go right ahead and run `npm i -g heroku`. It'll take a few seconds and then you're ready to roll out.

Next, you need to login. From your prompt, go ahead and run `heroku login`. This will ask you to enter your email, and password, for Heroku. 

Then finally, you need to create an *application* on Heroku, if you haven't already done that on the website. So in my case, I just went `heroku create eviebot` and that took care of that! Oh make sure you run this *in your project folder*.

Woop woop! Almost there! One last thing before put in the last puzzle pieces, which is to configure the same Environment Variables that we have in our `.env` file. Go right ahead and visit your dashboard, which would be the following URL: `https://dashboard.heroku.com/apps/<appname>/settings` (replace `<appname>` with the one you just created, of course). It should look like this: 

![Heroku Config Vars](https://img.evie.codes/QNSpgc4)

## Pushing the app

So we're on the last mile now! Everything you've done until now has prepared us for this moment. 

- `git init` make our project git-enabled. 
- `heroku create <appname>` generated a link to your heroku project. 
- `npm init -y` and the package.json file modifications prepared our project for the Heroku universe.

So let's finalize everything by saving our changes to `git` and pushing them to Heroku: 

- Add all the files to git with `git add .` (don't forget the dot!)
- Prepare for pushing using commit: `git commit -m "Initial bot commit for Heroku"`
- Push the entire thing to Heroku: `git push heroku master`

OMG. That's... it. can you believe it? Your bot's now on Heroku and should really be running already!

To verify, look at the logs, from the top-right "More" button: 

![Heroku Logs](https://img.evie.codes/hosTRYK)

It should look like this: 

![Heroku Success!](https://img.evie.codes/1Y6bJxZ)

So... here's to hoping for the best!