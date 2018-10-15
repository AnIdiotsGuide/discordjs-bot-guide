# `.env` Files and Environment Variables

`.env` files are a type of file that holds environment variables of an application. Environment variables allow you to easily integrate your bot with various online platforms (ex. Heroku), easily split your production and development environment as well as keep important information like your bot token, API tokens and database details secure.

You do not want this information to get out to anyone. And using environment variables in a `.env` file is one of the best ways to secure your information, being preferred over a `config.json` file.

To start out, it's suggested you install the `dotenv` package from NPM. This allows you to easily integrate environment variables into your bot. You may do that via the console. Make sure you are in your bot's root directory. Run this command:

`npm install dotenv`

Once installed, you will need to create a `.env` file. I'll create an example one here.
```
TOKEN=[YOUR_BOT_TOKEN]
OWNER=[YOUR_OWNER_ID]
PREFIX=[DEFAULT_BOT_PREFIX]
```
After you create and update this file, save it and head over to your main file. I will build off of the default Discord.js example code.
```js
const Discord = require('discord.js');
// Importing this allows you to access the environment variables of the running node process
require('dotenv').config();

const client = new Discord.Client();

// "process.env" access the environment variables for the running node process. TOKEN is the environment variable you defined in your .env file
const token = process.env.TOKEN;
const prefix = process.env.PREFIX;

client.on('ready', () => {
  console.log(`Logged in as ${client.user.tag}!`);
});

client.on('message', msg => {

  // Here's I'm using one of An Idiot's Guide's basic command handlers. Using the PREFIX environment variable above, I can do the same as the bot token below
  if (message.author.bot) return;
  if (message.content.indexOf(prefix.length) !== 0) return;

  const args = message.content.slice(prefix.length).trim().split(/ +/g);
  const command = args.shift().toLowerCase();

  if (command === 'ping') {
    msg.reply('Pong!');
  }
});

// Here you can login the bot with the environment variable you defined above
client.login(token);
```
This is all you will need to get off the ground. Now I will explain why this is important and why you should use environment variables with a `.env` file over a `config.json` file.

## Using Git (ex. GitHub)
If you're going to publish your code to the public, for example, a Heroku application, then it's imperative you secure information such as your bot token, other API tokens, and any database details. Setting a `.env` file will allow you to do this, but there is an extra step you're going to have to take. And that's using a `.gitignore` file. Simply add the `.env` file to the `.gitignore` file in your local repository. From there on out, any future commits will ignore that file. You can set up a `.env.example` file in its place, but that's not always necessary.
```
.env
```
To read more information on `.gitignore` files, read [Using Git to share and update code](https://anidiots.guide/other-guides/using-git-to-share-and-update-code#ignoring-files) here on An Idiot's Guide.

If you don't add your `.env` to the `.gitignore` file then you risk your bot token being compromised. Don't let it come to that. The proper security measures.

If you follow this properly, all users will see in your bot code is `process.env.ENV_VARIABLE` which exposes nothing!

## Using Heroku
Heroku is a website that allows you to start out and host your applications for free. In this case, your Discord bot. However, Heroku requires that you use environment variables. If you setup your files with the `dotenv` package and required the specific environment variables in your code, then all you have to do is go the environment variables in your Heroku application and add the key and value.

In Heroku, these variables are called "Config Vars". I'm not going to go in-depth here about Heroku, but the same way you setup your `.env` file, you would add in new environment variables in Heroku.

![Heroku Config Vars](https://i.imgur.com/MSmEO5K.png)

The only way your bot token can be exposed along with other environment variables is if you do one or more of these things:
- a) You accidentally expose your `.env` file because you didn't add it to `.gitigonre`
- b) You added a collaborator through Heroku. A collaborator has almost full permissions as the app owner

Both a and b can be controlled. It's just you need to be smart.

## More information
Here are some links to more information you can read regarding environment variables and Git if you aren't that familiar with it:
- Using Git to share and update code (An Idiot's Guide): https://anidiots.guide/other-guides/using-git-to-share-and-update-code#ignoring-files
- Working with Environment Variables in Node.js (Twilio Blog): https://www.twilio.com/blog/2017/08/working-with-environment-variables-in-node-js.html
- `dotenv` NPM: https://www.npmjs.com/package/dotenv
- `dotenv-flow` NPM (Used for multiple `.env` file): https://www.npmjs.com/package/dotenv-flow
