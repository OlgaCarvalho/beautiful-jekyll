---
layout: post
title: How to monitor Telegram
subtitle: Simple python tool
tags: [threatintel, tutorial, dev]
---

As a security researcher, it is our job to monitor threat actors activity and the forums where they interact.

In this post, I showcase how to create a simple script to get live data from Telegram channels.

  

## What we need

- Telegram account
- Subscribed channels: we will see below [how to choose Telegram channels to follow](#how-to-choose-telegram-channels-to-follow)
- Python to develop our app
- `telethon` Python package to interact with Telegram’s API


## What is Telegram?

[Telegram](https://telegram.org/) is a cloud-based and centralized instant messaging service, which is perfectly legitimate by itself.  

However, with its capability to create an anonymous environment and open groups up to 200.000 members, it became a safe house for threat actors to share their accomplishments, attacks and even recruit new members. Not only that, Telegram bots have been seen being used for phishing and malware delivery.

So, not surprisingly, Telegram has been gaining popularity among cybercriminals around the world, having become a branch of the dark web itself.

As such, our job as security researchers, is to use the same tool to gather threat intelligence, and that’s what we will be doing today.

  

### Is my phone number secure?

To sign up for a Telegram account you have to give the app your phone number and permission to make and receive calls.

However, Telegram does not share the number with other users name, making your username your identifying token, _unless_, you give it permissions to sync your contacts. Then, users will see your phone number if you have theirs stored in your phone.

  

To preserve further anonymity there is a way to hide your phone number on the app:

1. Open Telegram and tap the hamburger button on the top-left corner
2. Click on **Settings** and scroll down to **Privacy and Security**
3. On the **Privacy** section tap **Phone Number**
- On the section **Who can see my phone number?** select **Nobody** (no one will see your phone number)
- On the section **Who can find me by my number?** select the most restricting one. At this moment is **My Contacts**
- Go back one page
5. Still on the **Privacy** section you can see other different settings that you can further tune.

  

_Note_: Keep in mind that your phone number will always be attached to your account.

  

### Can I create a Telegram account without my phone number?

Yes, absolutely.

If you are in the U.S check out Google Voice. It is free there and it can give you a local telephone number. This number has to be connected to a Google Account, which is quite easy (and a good idea for OSINT) to set up a new account.

If you are in EU like me, unfortunately this service is not available (for free). But you can always use a burner phone, paid with cash so it doesn’t trace back to you.

  

### How to choose Telegram channels to follow

There is a popular repository of deep and dark web sources you can check out → [DeepdarkCTI](https://github.com/fastfire/deepdarkCTI/blob/main/telegram.md "https://github.com/fastfire/deepdarkCTI/blob/main/telegram.md").

If you need inspiration take a look at SOCRadar's blog on their [top 10 darkweb telegram chat groups and channels](https://socradar.io/the-top-10-dark-web-telegram-chat-groups-and-channels/ "https://socradar.io/the-top-10-dark-web-telegram-chat-groups-and-channels/").

  

  

## Tutorial

### Step 1. Create a Telegram application

To get started, we need an  `api_id` and an `api_hash`. 

To get these, login to your account using [https://my.telegram.org/](https://my.telegram.org/auth). 

Follow the instructions to sign in and go to **API development tools**.

Fill out the form to receive the parameters we are looking for.

  

If you get an “ERROR” when you try to **Create application**, here a few tips:

- **App title** and **Short name** should be alpha-numeric 5-32 characters
- Open the url, [https://my.telegram.org/](https://my.telegram.org/.), in incognito mode and without any VPN enabled

  

### Step 2. Python client

First, let’s create a file where we will store our access tokens and set them as environment variables. This environment variables then will be used by the python client to start the connection with Telegram.

  

Create a file `⁠telegram_secrets.py`⁠ and write the following:

```
import os

def set_environment_variables():
    os.environ["TELEGRAM_API_ID"] = "<your-api_id>"
    os.environ["TELEGRAM_API_HASH"] = "<your-api_hash>"
    os.environ["TELEGRAM_USERNAME"] = "<your-username>"

```

Use your access tokens and username, save and close.

  

Create another file for our python client, `⁠telegram-monitor.py`⁠. This will be the script that will actively monitor Telegram.

  

```python
import os
from telethon import TelegramClient, events
from telegram_secrets import set_environment_variables
from slack_sdk import WebClient

# Set and get environment variables
set_environment_variables()
api_id = os.environ.get("TELEGRAM_API_ID")
api_hash = os.environ.get("TELEGRAM_API_HASH")
username = os.environ.get("TELEGRAM_USERNAME")

# Keywords to filter results
keywords = []   # add here your keywords

# Initialize the client with your tokens
client = TelegramClient(username, api_id, api_hash)
client.start()

# Handle receiving a new message
@client.on(events.NewMessage())
async def message_handler(event):
    chat = await event.get_chat()
    user = await event.get_sender()
    try:
        if(event.raw_text):
            event.raw_text = event.raw_text.lower()
            if any(word in event.raw_text for word in keywords):
                print("*Chat*: " + chat.title)
                print("\n*User*: " + user.username)
                print("*Message*:\n" + event.raw_text)
    except:
        pass

# Run the client until it is disconnected
client.run_until_disconnected()     # use Ctrl+C to disconnect

```

  

Make sure (1) you understand the code above, and (2) add your own words to the list `⁠keywords`⁠ .

  

After you run the program you will see something like:
```powershell
Please enter your phone (or bot token):     # insert your phone number 
Please enter the code you received:     # check your Telegram app for a message with a code  
Signed in successfully as <your username>; remember to not break the ToS or you will risk an account ban!
```
  

Insert your phone number (with country indicator) and the code Telegram will send you in the app.

  

And voilá!

Happy Hunting 🕵️‍♀️