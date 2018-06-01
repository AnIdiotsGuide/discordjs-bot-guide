# Hosting on a Raspberry Pi

{% embed data="{\"url\":\"https://www.youtube.com/watch?v=kpci6V8969g\",\"type\":\"video\",\"title\":\"HOSTING EPISODE \#1: RASPBERRY PI\",\"description\":\"A lot of people ask how to host their bots on a raspberry pi, and there\'s a lot of conflicting information on the internet, well my fellow idiots, I\'m here to give you the information you\'ll need to get your discord.js bot running on node v6 on a raspberry pi.\\n\\nI\'m using a Raspberry Pi 2 Model B running in headless mode with a WiFi dongle, on Raspbian 8 \(Jessie\)\\n\\n\\nCommands used in the video;\\nlsb\_release -a\\nnode --version\\nnpm --version\\nsudo apt-get remove --purge node\* npm\*\\ncurl -sL https://deb.nodesource.com/setup\_6.x \| sudo -E bash -\\nsudo apt-get install -y nodejs\\nsudo npm install pm2 -g\\npm2 startup\\nThen follow the on screen instructions\",\"icon\":{\"type\":\"icon\",\"url\":\"https://www.youtube.com/yts/img/favicon\_144-vfliLAfaB.png\",\"width\":144,\"height\":144,\"aspectRatio\":1},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://i.ytimg.com/vi/kpci6V8969g/maxresdefault.jpg\",\"width\":1280,\"height\":720,\"aspectRatio\":0.5625},\"embed\":{\"type\":\"player\",\"url\":\"https://www.youtube.com/embed/kpci6V8969g?rel=0&showinfo=0\",\"html\":\"<div style=\\\"left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.2493%;\\\"><iframe src=\\\"https://www.youtube.com/embed/kpci6V8969g?rel=0&amp;showinfo=0\\\" style=\\\"border: 0; top: 0; left: 0; width: 100%; height: 100%; position: absolute;\\\" allowfullscreen scrolling=\\\"no\\\"></iframe></div>\",\"aspectRatio\":1.7778}}" %}

A lot of people have asked how to host their bots on a raspberry pi, and there's a lot of conflicting information on the internet, well my fellow idiots, I'm here to give you the information you'll need to get your discord.js bot running on node v6 on a raspberry pi.

I'm using a Raspberry Pi 2 Model B running in headless mode with a WiFi dongle, on Raspbian 8 \(Jessie\)

Commands used in the video;

`lsb_release -a` - This will show you what version of Raspbian you're running.

`node --version` - This will show you what version of Node you're running.

`npm --version` - This will show you what version of NPM you're running.

`sudo apt-get remove --purge node* npm*` - This will remove _**ALL**_ traces of Nodered.

> _**NOTE:**_ There has been conflicting information regarding the purge command, a small number of people have claimed it has messed up their Raspbian installation, so please _**USE AT YOUR OWN RISK**_, I cannot be held responsible if it does go wrong, as I encountered no issues running these commands in this order.

`curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -` - This will download everything required for node to be installed.

> _**NOTE:**_ You can swap out `setup_6.x` for `setup_8.x` to update straight to Node 8.

`sudo apt-get install -y nodejs` - This will install node for you, the `-y` flag will auto-accept all terms and agreements for you.

`sudo npm install pm2 -g` - This will install pm2 \(Process Manger 2\), this _isn't_ required, but it's a great thing to have to restart your bots if you lose power or crash.

`pm2 startup` - This will begin the process of creating a start up script.

Then follow the on screen instructions

