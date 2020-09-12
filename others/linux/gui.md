# GUI の話

Linux で GUI  を使っている時に関するメモ

## 環境
- Centos 7.x
- GNU

ファイルの場所は以下
- /usr/share/applications/
- /home/${USER}/.local/share/applications/

ファイルの内容
```
[Desktop Entry]
Version=1.0
Type=Application
Name=IntelliJ IDEA Ultimate Edition
Icon=/home/${USER}/opt/idea-IU-192.6603.28/bin/idea.svg
Exec="/home/${USER}/opt/idea-IU-192.6603.28/bin/idea.sh" %f
Comment=Capable and Ergonomic IDE for JVM
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-idea
```