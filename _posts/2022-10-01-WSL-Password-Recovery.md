---
layout: post
title: WSL Password Recovery
tags: Linux WSL Tips
---

I have been using WSL for some time now. Having quick and easy access to a nix distribution is so handy in many ways. Not having to worry about managing VMs in a hypervisor makes the whole experience so much nicer. Today I switched back to a Ubuntu install I hadn’t used from quite a while and the first thing was to run sum updates but I had forgotten the password I had set. 

This post covers the simple process to preform a password recovery of an WSL install.

<!--more-->


### 1. Switch to root
First thing is to change the default user for the distribution. This can be done by editing the WSL distributions configuration with a simple command on the windows command line. 

```powershell
PS C:\Users\user> ubuntu config --default-user root
```
Now when the distribution is launched we will be logged in as the ```root``` user.


### 2. Change the Password 
Now that we have root access to the distribution we can change our password like you would with any linux distribution. 

```bash
user@box:~/$ passwd my-user
New password:
```

### 3. Restore the default user
 Now that the password has been changed, don't forget to restore the default user. 
```powershell
PS C:\Users\user> ubuntu config --default-user my-user
```


That’s it! It’s really a quick and simple process and not to dissimilar to the process you would use if we were working with a physical of VM install of a Nix distribution.  