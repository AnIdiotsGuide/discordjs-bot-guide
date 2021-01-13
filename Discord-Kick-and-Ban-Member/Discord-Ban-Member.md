## Discord BAN User

- First you need to create config.json file
and then add your bot prefix and BOT_TOKEN

> your config.json file will look like this

![image](https://user-images.githubusercontent.com/65860180/104457180-b2b91f80-55cf-11eb-8d47-465b7c204b01.png)



- Now in your main file in my case it is index.js where we write our bot code, 
we have to write code to kick member from discord server

- Lets start with writing default code for initializing the bot
![image](https://user-images.githubusercontent.com/65860180/104459655-08db9200-55d3-11eb-841b-44c540d10a5f.png)
>prefix and token is imported from config.json file


> - We need to make sure the user who will type the command for kicking member should have rights i.e. permission to ban member.
> - So we will provide an if condition to check whether he/she has permission or not, if yes then bot will ban the member from the server.

- Our code will look like this :


![image](https://user-images.githubusercontent.com/65860180/104461935-f9aa1380-55d5-11eb-9446-8df01118a0c0.png)

> Continued code

![image](https://user-images.githubusercontent.com/65860180/104462136-2eb66600-55d6-11eb-9756-611828b163a9.png)


- Results
![image](https://user-images.githubusercontent.com/65860180/104461568-87d1ca00-55d5-11eb-9def-2990a3997bb9.png)




### Hope You understood now how to BAN members from server.

