# sambaファイルサーバー設定
sambaで家のファイルサーバーを立てたときのメモ  
実際はdockerコンテナでやったけど、設定ファイルがあんまりきれいにまとまらなかったので、sambaの設定だけメモしておく、osはalpine

## sambaインストール
```
apk update
apk add --no-cache samba
```

## samba設定
### `/etc/samba/smb.conf`
```
# グローバル設定
[global]
   workgroup = WORKGROUP
   server string = Samba Server
   server role = standalone server
   hosts allow = ipアドレス
   hosts deny = all
   guest account = nobody
   map to guest = Bad User
   log file = /usr/local/samba/var/log.%m
   max log size = 50
   dns proxy = no
   unix extensions = no
   unix charset = UTF-8
   dos charset = CP932

# 特定のユーザー用ディレクトリ(パスワード認証付き)
[homes]
   comment = Home Directories
   browseable = no
   writable = yes

# ゲストユーザー用ディレクトリ(lan内誰でも)
[share]
   path = /home/public
   guest ok = yes
   guest only = yes
   browsable = yes
   writable = yes
   create mask = 0777
   directory mask = 0777
```

### ユーザー作成
```
adduser -D ユーザ名
# パスワード作成
chpasswd
# samba用にユーザー設定
pdbedit -a -u ユーザ名
```

### ゲスト用のディレクトリ作成
```
mkdir /home/public
chmod 777 /home/public
```

### 実行
```
nmbd -D
smbd -FS --no-process-group --configfile=/etc/samba/smb.conf </dev/null
```

## その他
必要なポートは `139` と `445`
