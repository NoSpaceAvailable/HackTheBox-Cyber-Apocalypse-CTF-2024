# Challenge source file
URL: [https://1drv.ms/u/c/4ab09e7ce6bb4141/EfnB0NVOEgZIqaf6QMvvoO0BHgHempS9NSvwPksKHl5Ymg?e=eaOp1J](https://1drv.ms/u/c/4ab09e7ce6bb4141/EfnB0NVOEgZIqaf6QMvvoO0BHgHempS9NSvwPksKHl5Ymg?e=eaOp1J)
(This is a blackbox challenge. Please try to solve it without source code first)
# Difficulty
Very easy

# Author
Hack The Box team

# Approach
- When I connect to the site, a login panel appeared:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/500c9da1-bdbe-47dc-81ae-514b33bd3eca)

- I thought that this seems to be a SQL injection challenge, so I tried some payload to prove it:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/a9492363-c198-4c4f-9725-3fdf2bed4b76)

- I also tried to modify the payload to bypass login page but it didn't work. Maybe this challenge is an error-based or time-base SQL injection. For fast result, I used *SQLMap* to reveice information from database. If you dont want to use SQLMap, you can try my time-base payload to receive the admin password:
  ```
  admin' UNION SELECT CASE WHEN (SELECT SUBSTRING(password, 1, 1) FROM users) = 'z' THEN SLEEP(5) ELSE 1 END = 1 #
  ```

  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/bc84bef3-b876-4d0b-9654-21d5130d1d6b)

# Problem Solving
- This is a bcrypt password hash. I use *john* to crack it with wordlist:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/46edaaee-f56e-4476-a8e6-6e3286ffd0bd)

- Then re-login:
  
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/6aabeb09-de67-4648-8bd6-54384e820f98)

- Flag: HTB{t3rm1n4l_cr4ck1ng_sh3n4nig4n5}
