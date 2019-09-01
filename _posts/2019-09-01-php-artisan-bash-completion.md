---
layout: post
title:  "php artisan bash completion"
date:   2019-09-01 11:20:00 +0800
---

因為常常忘記 `php artisan` 的指令，需要自動補全(bash completion)  
以下附上網路上找到的解答
<!--more-->
<script src="https://gist.github.com/tuanpht/2c92f39c74f404ffc712c9078a384f39.js"></script>

另外在 `vim` 編輯 `~/.bash_profile` ，要貼上上面的 code 的時候  
可以先下 `:set paste`，把自動縮排關掉  


來源：<a href="https://stackoverflow.com/questions/2514445/turning-off-auto-indent-when-pasting-text-into-vim/18488630" target="_blank" rel="noopener">https://stackoverflow.com/questions/2514445/turning-off-auto-indent-when-pasting-text-into-vim/18488630</a>
