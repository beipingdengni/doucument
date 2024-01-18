macOS：在macOS上，Apache的安装目录可能在/etc/apache2/，而网站文件可能位于/Library/WebServer/Documents/。





## PhpWebStudy

https://github.com/xpf0000/PhpWebStudy



mac 处理没有权限问题



```
sudo codesign --sign "phpca" --force --keychain ~/Library/Keychains/login-keychain-db /opt/homebrew/Cellar/php@7.4/7.4.33_5/lib/httpd/modules/libphp7.so

sudo codesign --sign "cxca" --force --keychain ~/Library/Keychains/login.keychain-db /opt/homebrew/Cellar/php@7.4/7.4.33_5/lib/httpd/modules/libphp7.so

LoadModule php7_module /opt/homebrew/Cellar/php@7.4/7.4.33_5/lib/httpd/modules/libphp7.so "PhpCA"
```

