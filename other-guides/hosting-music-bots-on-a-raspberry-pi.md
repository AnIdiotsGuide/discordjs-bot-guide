---
description: >-
  If you have a decent Raspberry Pi, you can maybe host a small music bot on it.
  Here's what you need to do that.
---

# Hosting Music Bots on a Raspberry Pi

{% embed data="{\"url\":\"https://www.youtube.com/watch?v=02jt7l3eBK0\",\"type\":\"video\",\"title\":\"HOSTING EPISODE \#2: MUSIC ON A PI\",\"description\":\"EDIT: It\'s been brought to my attention that there\'s a typo, at 7:13 I say windows-built-tools, it should read;\\n\\nnpm i -g windows-build-tools\\n\\nIn the last episode, I showed you how to get a bot up and running on a raspberry pi, in this episode I show you how to get a music bot up and running on the same raspberry pi, and I go over why you shouldn\'t host a music bot on the raspberry pi.\\n\\nJust because you CAN do it, doesn\'t mean you SHOULD.\\n\\nI\'m using a Raspberry Pi 2 Model B running in headless mode with a WiFi dongle, on Raspbian 8 \(Jessie\)\",\"icon\":{\"type\":\"icon\",\"url\":\"https://www.youtube.com/yts/img/favicon\_144-vfliLAfaB.png\",\"width\":144,\"height\":144,\"aspectRatio\":1},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://i.ytimg.com/vi/02jt7l3eBK0/maxresdefault.jpg\",\"width\":1280,\"height\":720,\"aspectRatio\":0.5625},\"embed\":{\"type\":\"player\",\"url\":\"https://www.youtube.com/embed/02jt7l3eBK0?rel=0&showinfo=0\",\"html\":\"<div style=\\\"left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.2493%;\\\"><iframe src=\\\"https://www.youtube.com/embed/02jt7l3eBK0?rel=0&amp;showinfo=0\\\" style=\\\"border: 0; top: 0; left: 0; width: 100%; height: 100%; position: absolute;\\\" allowfullscreen scrolling=\\\"no\\\"></iframe></div>\",\"aspectRatio\":1.7778}}" %}

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

You will require a free discordapp.com account to use the discord service, sign up [here](https://discordapp.com/hypesquad?ref=PYisfiCTRf)

You can join the _"An Idiot's Guide Official Server"_ [here](https://discord.gg/gkZCQtH)

If you'd like to support me on Patreon, you can do so [here](https://www.patreon.com/anidiotsguide)

Don't forget to join the _"Official Discord.js Server"_ [here](https://discord.gg/bRCvFy9)

Check out the Discord Developers portal [here](https://discordapp.com/developers/docs/intro)

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

