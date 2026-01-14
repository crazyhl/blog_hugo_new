# Deepin 添加 Phpstorm 快捷方式到 Application 目录


首先创建 Phpstorm.desktop 文件，并把下面的代码输入

```shell
[Desktop Entry]
Categories=Development;
Comment[zh_CN]=
Comment=
Exec=/opt/phpstorm/bin/phpstorm.sh
GenericName[zh_CN]=IDE
GenericName=IDE
Icon=/opt/phpstorm/bin/webide.png
Name[zh_CN]=phpStorm　　　　　　　　　　　
Name=phpStorm　　　　　　　
Path=
StartupNotify=true
Terminal=false
Type=Application
X-DBUS-ServiceName=
X-DBUS-StartupType=
X-KDE-SubstituteUID=false
X-KDE-Username=Learn Programming
```

说明一下 Exec 是执行文件的路径，Icon 是图标的路径，Categories 是分类，这个根据系统来就可以了

然后执行下面两行代码 
```shell
sudo mv Phpstorm.desktop /usr/share/applications/
sudo chmod +x Phpstorm.desktop
```

这样就 ok 了，在 application 目录里面就会有 phpstorm了，就这么简单，这个适用于 debain 系列，其他应用也是这样添加就ok了

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/deepin-add-phpstorm-shortcut-to-the-application-directory/  

