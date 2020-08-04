# screen  

## 導入
サーバー側で設定
```shell
# yum install -y screen
```

## CLI コマンド
- screen 一覧  
screen -ls 
- screen の名前付き作成  
screen -S <name>
- screen にアタッチ（接続）  
screen -r <screen>  
screen -rx <screen>  


## screen でのコマンド
screen にアタッチしてからの操作コマンド
- 水平分割
Ctr + a => S
- 垂直分割
Ctr + a => |
- 画面移動
Ctr + a => tab
- 新規ウィンドウ
Ctr + c => c
- 分割スクリーンの削除
Ctr + a => x
