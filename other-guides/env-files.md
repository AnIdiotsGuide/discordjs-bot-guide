# Using Environment Variables

A `.env` file is a type of file that holds environment variables of an application. Environment variables allow you to easily integrate your bot with various online platforms \(ex. Heroku\), easily split your production and development environment as well as keep important information like your bot token, API tokens and database details secure.

You do not want this information to get out to anyone. And using environment variables in a `.env` file is one of the best ways to secure your information, being preferred over a `config.json` file.

To start out, it's suggested you install the `dotenv` package from NPM. This allows you to easily integrate environment variables into your bot. You may do that via the console. Make sure you are in your bot's root directory. Run this command:

`npm install dotenv`

Once installed, you will need to create a `.env` file. I'll create an example one here \(I will be using the `CLIENT_TOKEN` variable from the master branch of discord.js\).

_If using v12 branch of discord.js, use_ `DISCORD_TOKEN=`_._

```text
CLIENT_TOKEN=[YOUR_BOT_TOKEN]
OWNER=[YOUR_OWNER_ID]
PREFIX=[DEFAULT_BOT_PREFIX]
```

After you create and update this file, save it and head over to your main file. I will build off of the default Discord.js example code.

```javascript
const Discord = require('discord.js');
// Importing this allows you to access the environment variables of the running node process
require('dotenv').config();

const client = new Discord.Client();

// "process.env" accesses the environment variables for the running node process. PREFIX is the environment variable you defined in your .env file
const prefix = process.env.PREFIX;

client.on('ready', () => {
  console.log(`Logged in as ${client.user.tag}!`);
});

client.on('message', message => {

  // Here's I'm using one of An Idiot's Guide's basic command handlers. Using the PREFIX environment variable above, I can do the same as the bot token below
  if (message.author.bot) return;
  if (message.content.indexOf(prefix.length) !== 0) return;

  const args = message.content.slice(prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  if (command === 'ping') {
    message.reply('Pong!');
  }
});

// Here you can login the bot. It automatically attempts to login the bot with the environment variable you set for your bot token (either "CLIENT_TOKEN" or "DISCORD_TOKEN")
client.login();
```

This is all you will need to get off the ground. Now I will explain why this is important and why you should use environment variables with a `.env` file over a `config.json` file.

## Using Git \(ex. GitHub\)

If you're going to publish your code with Git to a site like GitHub, then it's imperative you secure information by making sure your `.env` isn't committed to a repository. You may do that using a `.gitignore` file. Simply add the `.env` file to the `.gitignore` file in your local repository.

From there on out, any future commits will ignore that file or any other files or directories in `.gitignore`. You can set up a `.env.example` file in its place, but that's not always necessary.

```text
.env
```

Following this, all users will see in your bot code is `process.env.ENV_VARIABLE` which exposes nothing!

## Using Heroku

Heroku is a website that allows you to start out and host your applications for free. In this case, your Discord bot. However, Heroku requires that you use environment variables. If you setup your files with the `dotenv` package and required the specific environment variables in your code, then all you have to do is go the environment variables in your Heroku application and add the key and value.

In Heroku, these variables are called "Config Vars". I'm not going to go in-depth here about Heroku, but the same way you setup your `.env` file, you would add in new environment variables in Heroku.

![Heroku Config Vars](https://i.imgur.com/MSmEO5K.png)

The only way your bot token can be exposed along with other environment variables is if you do one or more of these things:

* a\) You accidentally expose your `.env` file because you didn't add it to `.gitignore`
* b\) You added a collaborator through Heroku. A collaborator has almost full permissions as the app owner

Both a and b can be controlled. It's just you need to be smart.

## Set environment variables in the start script

When starting your application, either locally on your computer, in a npm start script in your `package.json` or even in a `Dockerfile` you can set what environment your bot should run in.

For example, you may want to run your bot in production on Heroku or Glitch and in development on your computer. You can simply do that via adding new scripts in your `package.json`.

Upon doing `npm init` when you first made your bot, you should have seen a `test` script created. The scripts portion of your `package.json` should look like this if you added nothing.

```javascript
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
},
```

Here, you can add multiple scripts. Where going to add a few scripts. `production`, `development`, and `start`.

```javascript
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "node .",
  "production": "NODE_ENV=production&&npm start",
  "development": "set NODE_ENV=development&&npm start"
},
```

* To start your bot in a production environment, you would do `npm run production`. This will set `process.env.NODE_ENV` to `production`
* To start your bot in a development environment, you would do `npm run development`. This will set `process.env.NODE_ENV` to `development`

In your code, you can define what should happen depending on the environment loaded. Here's an example where your bot should only show the stream status if the environment is in `production` when the `ready` event is fired:

```javascript
require('dotenv').config();

// process.env.NODE_ENV allows you to get the environment the node process is in
let ver = process.env.NODE_ENV;

client.on('ready', () => {

  if (ver === 'production') {
    client.user.setActivity('An Idiot\'s Guide', { type: 'STREAMING', url: 'https://twitch.tv/something' })
  } else {
    client.user.setActivity('in code land', { type: 'PLAYING' });
  }
});
```

Here I make sure that the bot is set to a streaming status only if my node environment is in `production`. I don't want my bot shown as streaming while I'm working on it in a development environment. There is a lot more you can do with this though.

For example, you can change the `production` and `development` scripts to use entirely different main files if you wish. But I won't get into that here.

If you don't want to use start scripts, you can always set the node environment directly in the command line. Here's how:

```bash
# Windows
SET NODE_ENV=development&&npm start
SET NODE_ENV=development&&node .
SET NODE_ENV=development&&node app.js

# Linux/MacOS
NODE_ENV=development&&npm start
NODE_ENV=development&&node .
NODE_ENV=development&&node app.js
```

Either running these commands directly through the command line or in start scripts should work. It's entirely up to you. But for ease of use when deploying your bot, you should use start scripts. So for example, in Heroku, you can add this in your `Procfile`:

```text
# Heroku will run the bot in production mode
worker npm run production
```

If you're using Glitch, you can add this in your `start` script in your `package.json`:

```javascript
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "NODE_ENV=production&&node app.js"
},
```

You may need to change the name of the main file depending on what you called it/where it's located in your bot project.

## More information

Here are some links to more information you can read regarding environment variables and Git if you aren't that familiar with it:

* [Using Git to share and update code \(An Idiot's Guide\)](https://anidiots.guide/other-guides/using-git-to-share-and-update-code#ignoring-files)
* [Working with Environment Variables in Node.js \(Twilio Blog\)](https://www.twilio.com/blog/2017/08/working-with-environment-variables-in-node-js.html)
* [`dotenv` NPM](https://www.npmjs.com/package/dotenv)
* [`dotenv-flow` NPM \(Used for multiple `.env` file\)](https://www.npmjs.com/package/dotenv-flow)

