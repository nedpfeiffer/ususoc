---
title: "HTB University CTF 2023: MSS Write-Up"
description: "ChatGPT is going to take all our jobs"
date: 2023-12-12
draft: false
tags: [cryptography]
categories: [write-ups]
---

### by Ned Pfeiffer

This past weekend I participated in the 2023 HackTheBox University challenge, which was a very humbling experience. Our team ultimately ended up finding only 2 of 22 total flags, struggling on the easiest challenges. Nevertheless, it was a fun learning experience and it's got me feeling motivated to continue learning.

I took a crack at the first reverse engineering problem, realized I'm a script kiddie, then settled on cryptography. After netcating into the server, I was greeted with the following:

```
#     #  #####   #####               #       ###   
##   ## #     # #     #             ##      #   #  
# # # # #       #                  # #     #     # 
#  #  #  #####   #####     #    #    #     #     # 
#     #       #       #    #    #    #     #     # 
#     # #     # #     #     #  #     #   ## #   #  
#     #  #####   #####       ##    ##### ##  ###

This is a secure secret sharing scheme with really small threshold. We are pretty sure the key is secure...
    

Send in JSON format any of the following commands.

    - Get your share
    - Encrypt flag
    - Exit
```

I floundered around for a bit, then asked ChatGPT for help. I provided the server.py file and asked how to properly format commands.

**Chat GPT**

**The error you encountered indicates that the server expects the command in a properly formatted JSON object. When you sent query = get_share, it was not in JSON format, hence the error.**

**To correct this, you should send your command as a JSON string. For example, to use the "Get your share" command, you need to send a JSON object with the command field set to "get_share" and include an x value (which is your user ID in this context). Here is an example of how the JSON command should be formatted:**

```
{
    "command": "get_share",
    "x": 1234
}
```

Right, that's how JSON works (it's been a while since I took Intro to Data Analytics alright). When I gave that a try, I got the following output:

```
query = {"command": "get_share","x": 1234}
{"approved": "True", "x": 1234, "y": 63072648313734256227998936374562235306337018025668098133123226928799731058631021766964119155547177088143401584304546817685043506372611387435656249732008613793779552855322}
```

Cool! We're sort of getting somewhere. After floundering around a bit more, I asked a teammate who actually understands cryptography for help. After taking a look at the problem, he realized that asking for the key share 0 could just reveal the secret key. I grabbed that information and passed it to Chat GPT, which then proceeded to absolutely floor me.

**Ned**

**Help me decrypt the CTF flag based on the following information.**

```
query = {"command":"get_share","x":0}
{"approved": "True", "x": 0, "y": 32762276646637889181960356091700986105118354044634516959235466934097700344873}

query = {"command":"encrypt_flag","x":32762276646637889181960356091700986105118354044634516959235466934097700344873} 

[+] Here is your encrypted flag : {"iv": "7c3fa0a5faad4b791de0d74eb0e817fe", "enc_flag": "9a389ce033a9496ec42b921a2fcc392d1fc08c10705437b57b27bebf79c15ba10c4cc3592dc096f48751993fbc26c560"}.
```

**ChatGPT**

**To decrypt the flag, you need to reconstruct the secret key used in the AES encryption. Given that this is a polynomial secret sharing scheme, you would typically need multiple shares to reconstruct the polynomial and find the secret key. However, in your case, you have only one share (y value for x = 0), which fortunately is the secret key itself because the polynomial's value at x = 0 is the constant term (the secret key).**

**Let's decrypt the flag using the information provided. We'll extract the secret key from the share, use it to generate the AES key, and then decrypt the flag using the provided IV and encrypted data.**

**The decrypted CTF flag is: HTB{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX}.**

**This decryption was successful due to the specific setup of the secret sharing scheme, where the share corresponding to x = 0 was effectively the secret key itself, used for the AES encryption of the flag.**

I was shocked. ChatGPT 4's improved data analysis capabilities are seriously impressive and a little scary. For context, here's the Python script it wrote to find the solution.

```
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
from hashlib import sha256
import base64

# Extracted data from the user's interaction
y_share = 32762276646637889181960356091700986105118354044634516959235466934097700344873
iv_hex = "7c3fa0a5faad4b791de0d74eb0e817fe"
encrypted_flag_hex = "9a389ce033a9496ec42b921a2fcc392d1fc08c10705437b57b27bebf79c15ba10c4cc3592dc096f48751993fbc26c560"

# Convert hex values to bytes
iv = bytes.fromhex(iv_hex)
encrypted_flag = bytes.fromhex(encrypted_flag_hex)

# The secret key is the y-share for x = 0 in the polynomial secret sharing scheme
secret_key = y_share

# Generate the AES key using SHA-256 on the secret key
aes_key = sha256(str(secret_key).encode()).digest()

# Decrypt the flag
cipher = AES.new(aes_key, AES.MODE_CBC, iv)
decrypted_flag_padded = cipher.decrypt(encrypted_flag)
decrypted_flag = unpad(decrypted_flag_padded, 16)

decrypted_flag.decode('utf-8')
```

After reading Chat GPT's code analysis I had a mild existential crisis, questioning my aspirations to join the cybersecurity industry in the face of such advanced AI capabilities. However, I realized that while ChatGPT may provide in-depth technical insights, the value I bring is indispensable. My role involves not just understanding the AI's input but critically evaluating it, integrating diverse information from the problem at hand, suggestions from teammates, and insights from the AI. My ability to contextualize, ask the right questions, and apply creative thinking in unstructured scenarios is something that AI cannot replicate.

That previous paragraph was writen by Chat GPT. I'm not going to be able to find a job after graduation, am I?