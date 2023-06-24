---
title: "Fixing Git Permission Denied (Public Key) in Bash Shell [Windows]"
datePublished: Sun Jun 12 2022 04:05:05 GMT+0000 (Coordinated Universal Time)
cuid: cl4as9f4w06i5r3nv2sas9hda
slug: fixing-git-permission-denied-public-key-in-bash-shell-windows
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1654959535032/UJPbbt2ZB.png
tags: bash, git, windows, shell, ssh

---

## Story
I have just finished a feature by the end of my work day, and am ready to push my pretty codes to the repo and about to enjoy an accomplished moment 🚀...

BUT out of no where, after I run `git push origin feature/my-awesome-code`, <br />my Bash shell yells at me with this 🤯:

![ssh-error-permission-deny.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654790259564/G2ghQ_DL-.png align="left")

This gets me thinking and checking my SSH key and stuffs... all are correct! <br />
What on earth is wrong!!! 😵 <br />
And who cares about Git GUI, because pros only use Git command here. 😝

After, and after some tiring searches, I finally found the root cause of this problem.

## Root Cause
> The root cause is that Bash shell does not know my HOME

Normally, in Bash shell if you run `cd ~`, it will goes to your home directory (e.g. `C:\users\username`) where you stored your SSH key, instead it shows this weird `/y` drive which I don't remember existed in my computer.

![Group 2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654959836412/-UfFCYkNg.png align="left")

and obviously, you list files in this `/y`, you got nothing, that's why it shows above <code>error permission denied (publicKey)</code>, simply means the public key is not found there.

## Solution
To solve it, you need to set environment variable `HOME` to let your Bash shell aware.

- Window search for "Edit the system environment variables", will lead you to this dialog
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654789814281/mXDKfRcVN.png align="left")

- Set `HOME` to your home directory where SSH key is stored.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654789940007/OBE1JJBeg.png align="left")

- OK, Apply, OK...

- Restart your bash shell for it to take effect

- You've done, you've solved it! 🎉

### Extra Catches
If it still does not work for you,
 - try to restart your computer after setting the environment variable (very most likely no need to do so, but just do otherwise 😂)
 - Bash shell already got the public key, but you yourself miss configuring it in your remote repository, etc... 🙄