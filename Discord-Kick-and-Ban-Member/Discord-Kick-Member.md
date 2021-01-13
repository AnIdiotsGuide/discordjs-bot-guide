## Discord Kick User

- First you need to create config.json file
and then add your bot prefix and BOT_TOKEN

> your config.json file will look like this

![image](https://user-images.githubusercontent.com/65860180/104457180-b2b91f80-55cf-11eb-8d47-465b7c204b01.png)



- Now in your main file in my case it is index.js where we write our bot code, 
we have to write code to kick member from discord server

- Lets start with writing default code for initializing the bot
![image](https://user-images.githubusercontent.com/65860180/104459655-08db9200-55d3-11eb-841b-44c540d10a5f.png)
> prefix and token is imported from config.json file


> - We need to make sure the user who will type the command for kicking member should have rights i.e. permission to kick member.
> - So we will provide an if condition to check whether he/she has permission or not, if yes then bot will kick the member out of the server or else the bot will reply with the message "sorry you donot have permission to kick"

- Our code will look like this :

![image](https://user-images.githubusercontent.com/65860180/104460049-9b7c3100-55d3-11eb-9617-b2e7cf582eaf.png)
> Continued code

![image](https://user-images.githubusercontent.com/65860180/104460172-bea6e080-55d3-11eb-9a48-b9c116250580.png)

- Results
![image](https://user-images.githubusercontent.com/65860180/104460488-2ceba300-55d4-11eb-9cc2-6985e1b9cd43.png)


###Hope You understood now how to KICK members from server.