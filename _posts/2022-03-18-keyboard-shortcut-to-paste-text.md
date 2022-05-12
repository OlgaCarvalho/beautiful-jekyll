---
layout: post
title: Setup a keyboard shorcut to paste text
subtitle: How to use a key combination to paste any text you want
tags: [organization, howto, linux]
---
There are quite a few situations where you surely have felt that you are writing the same thing over and over again.

* When you need to enter your email address (but you don't want your browser to store it);
* When you use a shared mailbox but you like to sign your name;
* etc.



In this quick post I show you how you can set a keyboard shortcut to paste a predetermined text of your choice.



### Setup

**Step 1.** Install both `xdotool` and `xclip`

```sh
sudo apt-get install xdotool xclip
```

**Step 2.** Open your Keyboard Shortcuts window

In Xubuntu go to to your start menu and write "Keyboard". Open **Keyboard \| Applications Shortcuts** and select **+ Add**.

**Step 3.** Add the following one-liner. Change `YOUR TEXT` with anything you want.

```sh
/bin/bash -c "sleep 0.2 && printf 'YOUR TEXT' | xclip -selection clipboard && xdotool key Control_L+v"
```

**Step 4.** Select the keys you want to use for the shortcut. Be careful to not create a conflict with other keyboard shortcuts already set.



**Note** this will update your clipboard with "YOUR TEXT", so if you were copying other text, you will loose that text and `Ctrl+V` will paste "YOUR TEXT". 

**However** this will not interfere with the mouse middle-click clipboard since they are different buffers (different clipboards). This means you are able to select a piece of text with your mouse and paste it with middle-click **AND** use your shortcut at the same time. 





### Example

In my case I want the following text to be pasted whenever I use the keys combination: `Super+O`

```tex
Best Regards,
Olga Carvalho
```

So my one-liner will be:

```sh
/bin/bash -c "sleep 0.2 && printf 'Best Regards, \nOlga Carvalho' | xclip -selection clipboard && xdotool key Control_L+v"
```

And my key combination will be: `Super + O`



Now, whenever I have my cursor in a textfield and use my shortcut, I'll have my signature quickly typed.