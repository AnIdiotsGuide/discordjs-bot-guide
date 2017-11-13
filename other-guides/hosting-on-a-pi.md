{{ "https://www.youtube.com/watch?v=kpci6V8969g" | video }}

A lot of people have asked how to host their bots on a raspberry pi, and there's a lot of conflicting information on the internet,
well my fellow idiots, I'm here to give you the information you'll need to get your discord.js bot running on node v6 on a raspberry pi.

I'm using a Raspberry Pi 2 Model B running in headless mode with a WiFi dongle, on Raspbian 8 (Jessie)

Commands used in the video;

`lsb_release -a` - This will show you what version of Raspbian you're running.

`node --version` - This will show you what version of Node you're running.

`npm --version` - This will show you what version of NPM you're running.

`sudo apt-get remove --purge node* npm*` - This will remove ***ALL*** traces of Nodered.
>***NOTE:*** There has been conflicting information regarding the purge command, a small number of people have claimed it has messed up their Raspbian installation, so please ***USE AT YOUR OWN RISK***, I cannot be held responsible if it does go wrong, as I encountered no issues running these commands in this order.

`curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -` - This will download everything required for node to be installed.
>***NOTE:*** You can swap out `setup_6.x` for `setup_8.x` to update straight to Node 8.

`sudo apt-get install -y nodejs` - This will install node for you, the `-y` flag will auto-accept all terms and agreements for you.

`sudo npm install pm2 -g` - This will install pm2 (Process Manger 2), this _isn't_ required, but it's a great thing to have to restart your bots if you lose power or crash.

`pm2 startup` - This will begin the process of creating a start up script.

Then follow the on screen instructions
