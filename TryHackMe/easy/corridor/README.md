# Corridor

## 📋 Challenge Info
- **Platform**: [TryHackMe](https://tryhackme.com)
- **Room**: [Corridor](https://tryhackme.com/room/corridor) 
- **Difficulty**: 🟢 Easy
- **Category**: 🌐 Web & Network
- **Tags**: `web`, `IDOR`, `hash-cracking`, `md5`

### Challenge Description
> You have found yourself in a strange corridor. Can you find your way back to where you came?
>
> In this challenge, you will explore potential IDOR vulnerabilities. Examine the URL endpoints you access as you navigate the website and note the hexadecimal values you find (they look an awful lot like a hash, don't they?). This could help you uncover website locations you were not expected to access.  

## 🔍 Step 1: Reconnaissance
### 1.1. Website Exploration
Given the easy difficulty level, we can assume standard web ports (80 for `HTTP`, 443 for `HTTPS`). Simply navigating to the IP address in a **browser** reveals the website.  In this case, the attack vector is the website. No need to find another vectors using `nmap`.  

Upon visiting `http://10.65.179.5`, we see an interactive image with multiple clickable "doors".

![Corridor](images/corridor.png)

### 1.2. Analyzing HTML Structure
Using browser Developer Tools (F12) or viewing page source, we discover an HTML image map:
```html 
<body>
    <img src="/static/img/corridor.png" usemap="#image-map">

    <map name="image-map">
        <area target="" alt="c4ca4238a0b923820dcc509a6f75849b" title="c4ca4238a0b923820dcc509a6f75849b" href="c4ca4238a0b923820dcc509a6f75849b" coords="257,893,258,332,325,351,325,860" shape="poly">
        <!-- -- 12 more similar area tags -->
    </map>
</body>
```
Each `<area>` tag contains an `href` attribute with a 32-character hexadecimal string - these are the hashes mentioned in the challenge description.

### 1.3. Hash Extraction
Using command-line tools to extract all hash values:
```bash 
curl 10.65.179.5 | grep '<area' | grep -o 'href="[^"]*"' | cut -d'"' -f2
```
> [!NOTE] 
> - `curl 10.65.179.5` - Get webpage (`curl` use `HTTP` by default)  
> - `grep '<area'` - Find lines containing <area tags  
> - `grep -o 'href="[^"]*"'` - Extract href attributes with their values  
> - `cut -d'"' -f2` - Get the value between quotes  

**Collected Hashes (13 total):**
```text
c4ca4238a0b923820dcc509a6f75849b
c81e728d9d4c2f636f067f89cc14862c
eccbc87e4b5ce2fe28308fd9f2a7baf3
a87ff679a2f3e71d9181a67b7542122c
e4da3b7fbbce2345d7772b0674a318d5
1679091c5a880faf6fb5e6087eb1b2dc
8f14e45fceea167a5a36dedd4bea2543
c9f0f895fb98ab9159f51fd0297e236d
45c48cce2e2d7fbdea1afc51c7c6ad26
d3d9446802a44259755d38e6d163e820
6512bd43d9caa6e02c990b0a82652dca
c20ad4d76fe97759aa27a0c99bff6710
c51ce410c124a10e0db5e4b97fc2af39
```

## 🔬 Step 2: Hash Analysis
### 2.1. Hash Type Identification
32 hexadecimal characters = likely MD5.
```bash
echo "c4ca4238a0b923820dcc509a6f75849b" | hash-identifier
# Or using haiti:
haiti c4ca4238a0b923820dcc509a6f75849b
```
Both tools confirm: **MD5**.

### 2.2. Hash Cracking
Using [CrackStation](https://crackstation.net/), discovered the hashes corresponded to numbers 1-13:
![CrackStation Result](images/crack-station.png)

## 🚀 Step 3: IDOR Exploitation
### 3.1. Pattern Discovered
In this case:
- URL pattern: `http://target/md5(number)`
- Numbers 1-13 were accessible
- No authorization checks present
- We can potentially access other numbers

### 3.2. Logical Deduction

> The challenge asks: "Can you find your way back to where you came?"

Logical thinking: If we're currently at doors 1-13 (the visible clickable areas), where did we come from? Number 0 (the starting point).

### 3.3. Calculating md5 of "0"
```bash 
echo -n 0 | md5sum
# Output: cfcd208495d565ef66e7dff9f98764da
```

### 3.4. Accessing the Hidden Endpoint
Constructed and visited the hidden URL:
```text
http://10.65.179.5/cfcd208495d565ef66e7dff9f98764da
```
## 🚩 Step 4: Flag Capture
The hidden endpoint revealed the flag:
```text
flag{2477ef02448ad9156661ac40a6b8862e} 
 ````
 
--- 

## 🛡️ Security Recomendations
- **Use Non-predictable Identifiers**: Implement UUIDs or random strings
- **Implement Proper Authorization**: Check user permissions for each resource access
- **Input Validation**: Validate and sanitize all user inputs
- **Rate Limiting**: Prevent brute-force enumeration attacks
