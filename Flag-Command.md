# Challenge source file
URL: [https://1drv.ms/u/c/4ab09e7ce6bb4141/EWvNCZk9HmxElEJjFAFGvf0B81Bflocm4FpfJ7TTelcNbQ?e=oAMDPM](https://1drv.ms/u/c/4ab09e7ce6bb4141/EWvNCZk9HmxElEJjFAFGvf0B81Bflocm4FpfJ7TTelcNbQ?e=oAMDPM)
(This is a blackbox challenge. Please try to solve it without source code)
# Difficulty
Very easy

# Author
Hack The Box team

# Approach
- The website is about an escape game. When you type "start" into game console, some options will appear for you to choose. The plot of the game depends on what you choose:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/1c7db52e-e297-466c-83a2-c19e867ef245)

- After enjoying the game, I opened dev tool to see if there is any piece of code here, and I saw an endpoint:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/3caa05df-74cb-4498-8ac5-5b1f64024148)

- This endpoint displays all the game's option, including a secret command:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/84ab2688-aa8d-4f7b-a255-b6b37b2509d7)

# Problem-solving
- Back to the game, restart it and enter the secret command:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/bc765ea2-353b-4c24-8277-f0afffe938c9)

- Flag: HTB{D3v3l0p3r_t00l5_4r3_b35t_wh4t_y0u_Th1nk??!}
