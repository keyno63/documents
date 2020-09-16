# GUI の話

Linux で GUI  を使っている時に関するメモ  

## 何をしたいか
Intellij IDEA のバージョンアップをしたら、  
GUIの「アプリケーション」のものも更新したい。
インストールしただけだとバージョンアップ前のものを参照している。

## 環境
- Centos 7.x
- GNU

## 設定
### ファイルの場所は以下
- /usr/share/applications/
- /home/${USER}/.local/share/applications/

### ファイルの内容
IDEA の場合は以下  
`/home/${USER}/.local/share/applications/jetbrains-idea.desktop`
```
[Desktop Entry]
Version=1.0
Type=Application
Name=IntelliJ IDEA Ultimate Edition
Icon=/home/${USER}/opt/idea-IU-202.6397.94/bin/idea.svg
Exec="/home/${USER}/opt/idea-IU-202.6397.94/bin/idea.sh" %f
Comment=Capable and Ergonomic IDE for JVM
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-idea
```
