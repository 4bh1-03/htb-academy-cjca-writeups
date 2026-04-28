# Web Requests

Created by: **4bh1-03**

“If the operating system is the engine of a machine, then Web Requests are the fuel lines of the modern internet. After spending time deep-diving into `Linux basics` and `Windows PowerShell` on `HackTheBox`, I realized there was a ‘middleman’ I hadn’t fully introduced myself to yet: the HTTP protocol.

Every time you click a link, submit a form, or refresh a page, a silent, lightning-fast conversation happens between your browser and a server. In this post, I’m kicking off my walkthrough of the **`HTB Web Requests`** module.

![](images/image_0.png)

We’re moving past the GUI to look at what’s actually happening under the hood — using tools like `cURL` and `DevTools` to intercept, deconstruct, and manipulate the data that makes the web run, so let’s get started and break down how the web actually talks.

---

# **Section 1 : HyperText Transfer Protocol (HTTP)**

> [**cURL**](https://curl.haxx.se/) (client URL) is a command-line tool and library that primarily supports HTTP along with many other protocols. This makes it a good candidate for scripts as well as automation, making it essential for sending various types of web requests from the command line, which is necessary for many types of web penetration tests.
> 

![](images/image_1.png)

If we ever need to read more detailed documentation, we can use **`man curl`** to view the full cURL manual page.

### **1. To get the flag, start the above exercise, then use cURL to download the file returned by ‘/download.php’ in the server shown above.**

**Given socket (host+port) was : `94.237.61.52:45239`**

```bash
curl -O 94.237.61.52:45239/download.php

General format : curl -O <socket>/download.php
```

Here is the command executed on the HTB terminal:

![](images/image_2.png)

**Answer :** `HTB{64$!c_cURL_u$3r}` 

---

# **Section 3 : HTTP Requests and Responses**

In our earlier examples with cURL, we only specified the URL and got the response body in return. However, cURL also allows us to preview the full HTTP request and the full HTTP response, which can become very handy when performing web penetration tests or writing exploits. To view the full HTTP request and response, we can simply add the **`-v`** verbose flag to our earlier commands, and it should print both the request and response

**Given server : `94.237.61.52:45239`** 

### **1. What is the HTTP method used while intercepting the request? (case-sensitive)**

```bash
curl -v 94.237.61.52:45239
```

Here is the command executed on the HTB terminal:

![](images/image_3.png)

**Answer :** `GET` 

### **2. Send a GET request to the above server, and read the response headers to find the version of Apache running on the server, then submit it as the answer. (answer format: X.Y.ZZ)**

The `-i` flag includes only the protocol response headers in the output.

```bash
curl 94.237.61.52:45239 -i
```

Here is the command executed on the HTB terminal:

![](images/image_4.png)

**Answer :** `2.4.41` 

---

# **Section 4: HTTP Headers**

**Given server : `94.237.49.88:38467`** 

### **1. The server above loads the flag after the page is loaded. Use the Network tab in the browser devtools to see what requests are made by the page, and find the request to the flag.**

![](images/image_5.png)

![](images/image_6.png)

**Answer :** `HTB{p493_r3qu3$t$_m0n!t0r}`

---

# **Section 6 : GET**

### **HTTP Basic Auth**

```bash
curl -u username:password http://<SERVER_IP>:<PORT>/
```

This command uses **curl**, a command-line tool, to transfer data from or to a server. Specifically, it is performing a basic **HTTP GET request** with authentication.

**Breakdown of the Components**

**`curl`**: The name of the utility being used to communicate with the server.

- **`u username:password`**: This flag (short for `-user`) provides the **credentials** for Basic Authentication. It tells the server who is making the request so it can grant access to protected resources.

**`http://<SERVER_IP>:<PORT>/`**: This is the **URL** of the destination.

- `<SERVER_IP>`: The address of the server you're contacting.
- `<PORT>`: The specific "doorway" on the server where the service is listening (e.g., 8080 or 443).
- `/`: The root path of the server.

**What happens when you run it?**

When executed, **`curl`** sends a request to the server including an **`Authorization`** header. The server verifies the username and password; if they are correct, it will return the content of the page (usually HTML, JSON, or XML) directly to your terminal screen.

### **1. The exercise above seems to be broken, as it returns incorrect results. Use the browser devtools to see what is the request it is sending when we search, and use cURL to search for ‘flag’ and obtain the flag.**

**Given server : `94.237.63.176:34981`**

```bash
curl 'http://admin:admin@94.237.63.176:34981/search.php?search=flag' -v
```

Here is the command executed on the HTB terminal:

![](images/image_7.png)

**Answer :** `HTB{curl_g3773r}` 

---

# **Section 7 : POST**

**Given target : `94.237.61.202:37053`**

### **1. Obtain a session cookie through a valid login, and then use the cookie with cURL to search for the flag through a JSON POST request to ‘/search.php’**

First login to the website and get the session cookie from the response header. Then using the session cookie run the below request :

```bash
curl -X POST -d '{"search":"flag"}'
-b 'YOUR_SESSION_ID'
-H 'Content-Type: application/json'
'http://94.237.61.202:37053/search.php'
```

**Anatomy of the Request**

To understand how this interacts with the server, we can break it down into four key components:

- **The Method (`X POST`):** By default, `curl` performs a GET request. We use `X POST` to tell the server we are "posting" data to be processed—in this case, a search query.
- **The Payload (`d`):** This is the **data** we are sending. The string `'{"search":"flag"}'` is a JSON object. We are asking the `search.php` script to look for the term "flag."
- **Session Management (`b`):** The `b` (cookie) flag includes a **PHP Session ID**. This is crucial because it proves to the server that we are an active, authenticated user. Without this, the server might redirect us to a login page.
- **The Header (`H`):** We use a header to set the `Content-Type` to `application/json`. This informs the web server that the raw data in the body is JSON, allowing the PHP backend to decode it correctly.

**Why this matters**

This specific syntax is frequently used in **penetration testing** and **API debugging**. It allows you to simulate a front-end application’s behavior directly from the terminal, making it easier to inspect raw responses and headers that a browser might hide.

Here is the command executed on the HTB terminal:

![](images/image_8.png)

**Answer :** `HTB{p0$t_r3p34t3r}` 

---

# **Section 8 : CRUD API**

### **1. First, try to update any city’s name to be ‘flag’. Then, delete any city. Once done, search for a city named ‘flag’ to get the flag.**

```bash
curl -X PUT 'http://94.237.121.111:40125/api.php/city/London' -d '{"city_name":"flag"}' -H 'Content-Type:application/json'

curl -X DELETE 'http://94.237.121.111:40125/api.php/city/Boston'

curl -s 'http://94.237.121.111:40125/api.php/city/flag' | jq
```

Here are the commands executed on the HTB terminal:

![Updating city name to ‘flag’](images/image_9.png)

Updating city name to ‘flag’

![Deleting a random city](images/image_10.png)

Deleting a random city

![Getting city named ‘flag’ to get the flag](images/image_11.png)

Getting city named ‘flag’ to get the flag

**Answer :** `HTB{crud_4p!_m4n!pul4t0r}`

<aside>
<img src="https://www.notion.so/icons/light-bulb_green.svg" alt="https://www.notion.so/icons/light-bulb_green.svg" width="40px" />

**Note:** Make sure to not include spaces while mentioning the key:value pairs, else you will get a bad request error message.

</aside>

---

# **My Key Takeaways**

- **Browser DevTools: The X-Ray Vision.** I learned that the **`Network`** tab is the best place to start. It allows you to intercept what the browser is doing behind the scenes — seeing every cookie, header, and payload being sent in real-time.
- **cURL: The Command-Line Powerhouse.** While the browser is great for visualization, **`curl`** is where the real control happens. It allows for precise manipulation of requests (like custom headers and specific HTTP methods) that are difficult or impossible to trigger through a standard UI.
- **The Connection:** I realized that every button click in a browser is just a wrapper for a request. By capturing a request in DevTools and “Copying as cURL,” I can take that request into my terminal to automate it or test for vulnerabilities.

---