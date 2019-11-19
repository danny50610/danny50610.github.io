---
layout: post
title:  "php artisan bash completion"
date:   2019-09-01 11:20:00 +0800
tags:   [laravel]
---

因為常常忘記 `php artisan` 的指令，需要自動補全(bash completion)  
以下附上網路上找到的解答
<!--more-->
<script src="https://gist.github.com/tuanpht/2c92f39c74f404ffc712c9078a384f39.js"></script>

另外用 `vim` 進行**貼上**的動作時，可能會有**自動縮排**  
可以先下 `:set paste`，把**自動縮排**關掉  

## 2019-09-30 更新
最近發現 `.bashrc` 沒有被執行  
後來發現同時有 `.bashrc` 與 `~/.bash_profile` 的狀況下  
用 SSH 登入時，BASH 只會執行 `~/.bash_profile`  
所以要在 `~/.bash_profile` 最後加上  
```
if [ -f ~/.bashrc ]
then
    source ~/.bashrc
fi
```
或是可以嘗試寫在 `.bashrc`？

來源：
* <a href="https://stackoverflow.com/questions/2514445/turning-off-auto-indent-when-pasting-text-into-vim/18488630" target="_blank" rel="noopener">https://stackoverflow.com/questions/2514445/turning-off-auto-indent-when-pasting-text-into-vim/18488630</a>
* <a href="https://www.arthurtoday.com/2015/04/difference-between-bashrc-and-bash-profile.html" target="_blank" rel="noopener">https://www.arthurtoday.com/2015/04/difference-between-bashrc-and-bash-profile.html</a>