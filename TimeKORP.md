# Challenge source file
URL: [https://1drv.ms/u/c/4ab09e7ce6bb4141/EcJzLdklABNPlSsqQOtNZJEBhuYPRrm2uG6ihG6EVp6sYg?e=GeNxDq](https://1drv.ms/u/c/4ab09e7ce6bb4141/EcJzLdklABNPlSsqQOtNZJEBhuYPRrm2uG6ihG6EVp6sYg?e=GeNxDq)

# Difficulty
Very easy

# Author
Hack The Box team

# Approach
- The website is no more than a normal clock.
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/00484457-ef83-4753-99f8-7270c6dd247f)

- I tried to click on the *What's the date* banner and see that sonething was sent to server:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/7841c6cd-68fb-4365-9344-b141a798a066)
  
- After some fuzzing attempt, It appears that I can perform command injection on the site:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/a935fe25-0e65-49ff-aec4-bf8136484afb)

# Problem-solving
- Examining the source code, I found a piece of code that explained why the command was rejected:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/6e42e4a2-5b81-4288-84cf-ea48332f2764)

- The command *2>&1* will redirect all error message to standart output, so my first payload *'; ls;'* will divide the command into 3 part, and 2>&1 is a separate part. Maybe current user do not have permission to do such a thing, so I try this:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/7a6bc04e-0440-43e8-ad04-561bfc5c0268)

- It worked. Let's *Cat The Flag*:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/485795ad-1927-4f5b-a9f8-0b078adb8d78)

- Flag: HTB{t1m3_f0r_th3_ult1m4t3_pwn4g3}
