github被墙，所以如果想使用梯子访问github，建议通过以下方法。

一、HTTP形式

```
//proxy only for github

git config --global http.https://github.com.proxy "http://127.0.0.1:8080" 

// global proxy of http

git config --global http.proxy "http://127.0.0.1:8080" 

git config --global https.proxy "http://127.0.0.1:8080"

//global proxy of socks5

git config --global http.proxy "socks5://127.0.0.1:1080" 

git config --global https.proxy "socks5://127.0.0.1:1080"

//cancel proxy

git config --global --unset http.proxy 

git config --global --unset https.proxy
```

二、SSH形式

```
Host github.com
    Port 443 //must use 443 instead of 22
    User git
    HostName ssh.github.com
    IdentityFile "~/.ssh/id_rsa"
    ProxyCommand nc -X 5 -x 127.0.0.1:7890 %h %p
```

