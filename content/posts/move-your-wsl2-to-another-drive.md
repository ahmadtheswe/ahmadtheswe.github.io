+++
date = '2023-11-13T17:10:55+07:00'
draft = false
title = 'Move Your WSL2 to Another Drive'
author = 'ahmad'
categories = ['wsl', 'windows', 'linux']
tags = ['wsl', 'windows', 'linux']
featuredImage = '/images/move-your-wsl2-to-another-drive/featured.png'
+++
Windows Subsystem Linux (WSL) is one of the most useful modern tools for software development, in my opinion. Since Windows offered this feature, we don't need to use fully virtualized Linux (using VMWare or VirtualBox) but can just use WSL that fully integrated with our Windows machine.

The downside of WSL is we can't choose the directory to install it. I will installed by default in our system drive (C drive). This maybe not a problem if we allocate huge amount of storage for our C drive. But, the case is we commonly not allocate much storage for our system drive because we thought we will only use this drive to install essential things.

But there is a solution for this, you can follow this steps :

### 1st step, Install Linux Distro
Assume you already install any Linux distro from Microsoft Store. In this example, I will use Ubuntu (you can use whatever you own). Let's say I installed Ubuntu 22.04 in my Windows machine. I will get my installed distro on the list if I execute `wsl -l -v` command like below.

```powershell
PS> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-22.04    Running         2
```

### 2nd Export WSL instance
In this example, I will move my WSL instance (Ubuntu) to my E drive. You can open your PowerShell and run these commands :

```powershell
PS> cd E:
PS> mkdir wsl2
PS> cd wsl2
PS> mkdir Ubuntu-22.04
PS> wsl --export Ubuntu-22.04 .\Ubuntu-22.04\ext4.vhdx --vhd
PS> wsl --unregister Ubuntu-22.04
PS> wsl --import-in-place Ubuntu-22.04 .\Ubuntu-22.04\ext4.vhdx
```

Let me explain these commands :

* `wsl --export` is used to export the instance of Ubuntu-22.04 to ext4.vhdx inside Ubuntu-22.04 directory.

* `wsl --unregister` is used to unregister existing WSL2 with name Ubuntu-22.04.

* `wsl --import-in-place` is used to register/import instance that we just export (ext4.vhdx) as new Ubuntu-22.04.


### 3rd step, Set default user.

After importing the new instance, you will use root user by default. To change this to our user, we need to modify `/etc/wsl.conf` inside our WSL instance by adding this line :

```bash
[user]
default=<your-username>
```

Change `<your-username>` with username that you want to use.

That's all, really simple :)

---

External source :

* [Move WSL to Another Drive](https://blog.iany.me/2020/06/move-wsl-to-another-drive/)

* [Build 18980](https://learn.microsoft.com/en-us/windows/wsl/release-notes#build-18980)