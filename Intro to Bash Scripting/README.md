# Introduction to Bash Scripting

Created by: **4bh1-03**

Hello everyone! After working through the **`Linux Fundamentals module`** on Hack The Box Academy, I’m moving on to the next step in the **`Junior Cybersecurity Analyst path`**: **`Introduction to Bash Scripting`**.

![](images/image_0.png)

Just like last time, I’ve decided to create a detailed walkthrough of the module’s interactive sections. My goal is to help anyone else working through the path and to reinforce my own learning by teaching. Bash is a crucial tool for automation and system administration, so I’m excited to dive in!

Let’s get started with the first section.

---

# **Section 1 : Conditional Execution**

**Exercise Script :**

```bash
#!/bin/bash
# Count number of characters in a variable:
#     echo $variable | wc -c

# Variable to encode
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}
do
        var=$(echo $var | base64)
done
```

### **1. Create an “If-Else” condition in the “For”-Loop of the “Exercise Script” that prints you the number of characters of the 35th generated value of the variable “var”. Submit the number as the answer.**

**Solution Script :**

```bash
#!/bin/bash
# Count number of characters in a variable:
#     echo $variable | wc -c
# Variable to encode

var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}
do
        var=$(echo $var | base64)

        if [ $counter -eq 35 ]
        then
            echo $var | wc -c
        fi
done
```

This script repeatedly **Base64-encodes** the value stored in the variable `var` a total of 40 times. In each loop iteration, the variable is re-encoded. When the loop **counter reaches 35**, it prints the **number of characters** in the encoded string using echo **`$var | wc -c`** .

**Answer :** `1197735`

---

# **Section 2 : Arguments, Variables, and Arrays**

**Arrays.sh :**

```bash
#!/bin/bash

domains=("www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com"
www2.inlanefreight.com)
echo ${domains[0]}
```

### **1. Submit the echo statement that would print “www2.inlanefreight.com” when running the last “Arrays.sh” script.**

**Solution Script :**

```bash
#!/bin/bash

domains=("www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com"
www2.inlanefreight.com)
echo ${domains[1]}
```

**Answer :** `echo ${domains[1]}` .

---

# **Section 3 : Comparison Operators**

**Exercise Script :**

```bash
#!/bin/bash

var="8dm7KsjU28B7v621Jls"
value="ERmFRMVZ0U2paTlJYTkxDZz09Cg"

for i in {1..40}
do
        var=$(echo $var | base64)

        #<---- If condition here:
done
```

### **1. Create an “If-Else” condition in the “For”-Loop that checks if the variable named “var” contains the contents of the variable named “value”. Additionally, the variable “var” must contain more than 113,450 characters. If these conditions are met, the script must then print the last 20 characters of the variable “var”. Submit these last 20 characters as the answer.**

**Solution Script :**

```bash
#!/bin/bash

var="8dm7KsjU28B7v621Jls"
value="ERmFRMVZ0U2paTlJYTkxDZz09Cg"

for i in {1..40}
do
        var=$(echo $var | base64)

        if [[ $var == *$value* && ${#var} -gt 113450 ]]
        then
                echo $var | tail -c 20
        fi
done
```

The `tail -c 20` command displays the last 20 bytes of its input. However, it does **not visually display newline characters** (`\n`). Therefore, even though 20 bytes are printed, only **19 visible characters** appear on screen — the final (20th) byte is a **newline** character.

**Answer :** `2paTlJYTkxDZz09Cg==` 

---

# **Section 4 : Flow Control — Loops**

**Exercise Script :**

```bash
#!/bin/bash

# Decrypt function
function decrypt {
    MzSaas7k=$(echo $hash | sed 's/988sn1/83unasa/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/4d298d/9999/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/3i8dqos82/873h4d/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/4n9Ls/20X/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/912oijs01/i7gg/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/k32jx0aa/n391s/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/nI72n/YzF1/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/82ns71n/2d49/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/JGcms1a/zIm12/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/MS9/4SIs/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/Ymxj00Ims/Uso18/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/sSi8Lm/Mit/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/9su2n/43n92ka/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/ggf3iunds/dn3i8/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/uBz/TT0K/g')

    flag=$(echo $MzSaas7k | base64 -d | openssl enc -aes-128-cbc -a -d -salt -pass pass:$salt)
}

# Variables
var="9M"
salt=""
hash="VTJGc2RHVmtYMTl2ZnYyNTdUeERVRnBtQWVGNmFWWVUySG1wTXNmRi9rQT0K"

# Base64 Encoding Example:
#        $ echo "Some Text" | base64

# <- For-Loop here

# Check if $salt is empty
if [[ ! -z "$salt" ]]
then
    decrypt
    echo $flag
else
    exit 1
fi
```

### **1. Create a “For” loop that encodes the variable “var” 28 times in “base64”. The number of characters in the 28th hash is the value that must be assigned to the “salt” variable.**

**Solution Script :**

```bash
#!/bin/bash

# Decrypt function
function decrypt {
    MzSaas7k=$(echo $hash | sed 's/988sn1/83unasa/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/4d298d/9999/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/3i8dqos82/873h4d/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/4n9Ls/20X/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/912oijs01/i7gg/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/k32jx0aa/n391s/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/nI72n/YzF1/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/82ns71n/2d49/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/JGcms1a/zIm12/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/MS9/4SIs/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/Ymxj00Ims/Uso18/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/sSi8Lm/Mit/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/9su2n/43n92ka/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/ggf3iunds/dn3i8/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/uBz/TT0K/g')

    flag=$(echo $MzSaas7k | base64 -d | openssl enc -aes-128-cbc -a -d -salt -pass pass:$salt)
}

# Variables
var="9M"
salt=""
hash="VTJGc2RHVmtYMTl2ZnYyNTdUeERVRnBtQWVGNmFWWVUySG1wTXNmRi9rQT0K"

for counter in {1..28}
do
 var=$(echo $var | base64)
done

salt=$(echo $var | wc -m)

if [[ ! -z "$salt" ]]
then
    decrypt
    echo $flag
else
    exit 1
fi
```

A **for loop** is used to **Base64-encode** the variable **`var`** (`"9M"`) **28 times**. After the 28th encoding, the script counts how many characters are in the resulting string using **`wc -m`**, and assigns that number as the **`salt`** value.

> ***Why it is better to use `wc -m` instead of `wc -c` ?***
> 

The simple answer is:

- `wc -c` counts **bytes** (the raw data).
- `wc -m` Counts every **single character**, including letters, numbers, symbols, and whitespace (like newlines and tabs).

**Answer :** `HTBL00p5r0x`

---