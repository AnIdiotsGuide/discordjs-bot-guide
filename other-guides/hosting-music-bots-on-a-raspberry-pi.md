# Hosting Music Bots on a Raspberry Pi

{% embed url="https://www.youtube.com/watch?v=02jt7l3eBK0" %}

{% hint style="info" %}
It's been brought to my attention that there's a typo, at 7:13 I say windows-built-tools, it should read;

`npm i -g windows-build-tools` OR `npm i -g ffmpeg-binaries`
{% endhint %}

In the last episode, I showed you how to get a bot up and running on a raspberry pi, in this episode I show you how to get a music bot up and running on the same raspberry pi, and I go over why you shouldn't host a music bot on the raspberry pi.

Just because you **CAN** do it, doesn't mean you **SHOULD**.

I'm using a Raspberry Pi 2 Model B running in headless mode with a WiFi dongle, on Raspbian 8 \(Jessie\)

Commands used in the video;

`npm install` - This will install the bot from the package.json file.

`sudo apt-get install libav-tools -y` - This will install an FFMPEG compatible transcoder.

`pm2 start app.js --name Music` - This will add the music bot to PM2's list, and starts it up.

`pm2 list` - This will list all currently stored items.

`pm2 monit` This will pull up a live monitor so you can watch the resources.

_**LINK DUMP**_

AoDude's OhGodMusicBot is available [here](https://github.com/bdistin/OhGodMusicBot)

You will require a free discord.com account to use the discord service, sign up [here](https://discord.com/)

If you'd like to support me on Patreon, you can do so [here](https://www.patreon.com/anidiotsguide)

Check out the Discord Developers portal [here](https://discord.com/developers/docs/intro)

Discord.js Official Documentation is available [here](https://discord.js.org/#!/)

Don't forget, if you want to invite your bots to server, you'll need to get an invite link, you can use the Discord Permissions Calculator [here](https://finitereality.github.io/permissions/?v=0) to set up your permissions, and generate an invite link.

List of Atom addons and themes I use; Addons:

* atom-beautify - Glavin001,
* highlight-selected - richrace,
* linter-eslint - AtomLinter,
* seti-icons - wyze and
* tokamak-terminal - vertexclique

Themes:

* seti-syntax, seti-ui - jesseweed

