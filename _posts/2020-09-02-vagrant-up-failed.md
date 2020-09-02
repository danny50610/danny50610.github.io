---
layout: post
title:  "vagrant up 啟動失敗的暫時解法"
date:   2020-09-02 21:39:00 +0800
tags:   [Vagrant, VirtualBox]
---

如果你在 `vagrant up` 遇到奇怪的錯誤訊息，而且明明之前還正常  
而且最近剛好有跑 **Windows 大更新**的話

<!--more-->

**會建議你重新安裝 VirtualBox 和 Vagrant**

已經不曉得遇過多少次了  
而且沒有一次是真正解決問題的，重新安裝還比較快  
建議先從 VirtualBox 開始，接下來才是 Vagrant  
要先移除再安裝，如果需要重開機就重開機  

P.S. 我只有在 Vagrant 上面使用 [laravel/homestead](https://github.com/laravel/homestead)，不確定其他 box 會不會有這問題

底下是遇到的錯誤訊息
```
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["startvm", "2ca72a0b-994b-4bc2-8f8b-0cf364505f25", "--type", "headless"]

Stderr: VBoxManage.exe: error: Failed to open/create the internal network 'HostInterfaceNetworking-VirtualBox Host-Only Ethernet Adapter #2' (VERR_INTNET_FLT_IF_NOT_FOUND).
VBoxManage.exe: error: Failed to attach the network LUN (VERR_INTNET_FLT_IF_NOT_FOUND)
VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component ConsoleWrap, interface IConsole
```
```
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["startvm", "2ca72a0b-994b-4bc2-8f8b-0cf364505f25", "--type", "headless"]

Stderr: VBoxManage.exe: error: The virtual machine 'homestead' has terminated unexpectedly during startup with exit code 1 (0x1).  More details may be available in 'C:\Users\danny\VirtualBox VMs\homestead\Logs\VBoxHardening.log'
VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component MachineWrap, interface IMachine

```