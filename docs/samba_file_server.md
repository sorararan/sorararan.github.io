# sambaファイルサーバー設定
sambaで家のファイルサーバーを立てたときのメモ

## 構成
環境をあまり汚さないようにdockerでやることにした
```
file_server:
  - .env
  - docker-compose.yml
  - Dockerfile
  - run.sh
  - smb.conf
```

## docker関連
### `Dockerfile`
ラズパイで動かす際にarmプロセッサ用の `samba` イメージがあるのか知らんので `alpine` からイメージを作った
```
FROM alpine:latest

RUN apk update \
  && apk add --no-cache samba

COPY ./smb.conf /etc/samba/smb.conf
COPY ./run.sh /usr/local/bin/run.sh
RUN chmod 500 /usr/local/bin/run.sh

EXPOSE 139 445
CMD ["/bin/ash", "-c", "/usr/local/bin/run.sh"]
```

### docker-compose.yml
ローカルでマウントするところのパスのところはよしなに  
(うちでは外付けhddを当てた)
```
version: "3"
services:
  samba:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "139:139"
      - "445:445"
    env_file:
      - ./.env
    volumes:
      - ローカルでマウントするところのパス:/home
    restart: always
```

## samba関連
### `smb.conf`
```
# グローバル設定
[global]
   # windowsクライアントと同じワークグループを指定する
   workgroup = WORKGROUP
   # ネットワークコンピュータ一覧を表示した際に、表示される内容らしい
   server string = Samba Server
   # 動作モード設定
   server role = standalone server
   # アクセスを許可するipアドレス
   hosts allow = 192.168.3.
   # 許可したipアドレス以外は拒否
   hosts deny = all
   # ゲストアカウント指定
   guest account = nobody
   # 指定されたユーザがないとき、ゲストでログイン許可
   map to guest = Bad User
   # windowsクライアントで対応してないunix拡張off
   unix extensions = no

# 各ユーザー専用のディレクトリ
[homes]
   # 説明欄に表示される
   comment = Home Directories
   # ネットワークコンピュータに表示しない
   browseable = no
   # ファイル書き込みできる
   writable = yes

# 共有ディレクトリ
[share]
  # 割り当てるパス指定
   path = /home/public
   # ゲストアクセス許可
   guest ok = yes
   # ファイル操作はguestによって実行
   guest only = yes
   # ネットワークコンピュータに表示する
   browsable = yes
   # ファイル書き込みできる
   writable = yes
   # shareに上げた場合はもうrwx何でも
   create mask = 0777
   directory mask = 0777
```

### `run.sh`
```
#!/bin/ash
chmod 777 /home/public

ls -l /home/ | grep ^d | grep -v public | awk '{print $9}' | while read username
do
  adduser -DH $username
  echo "$username:`eval echo '$'$username`" | chpasswd
  echo -e "`eval echo '$'$username`\n`eval echo '$'$username`" | pdbedit -a -u $username
done

nmbd -D
smbd -FS --no-process-group --configfile=/etc/samba/smb.conf </dev/null
```

### `.env`
`.gitignore` が効くように、パスワードはこのファイルに分離した  
以下のような形式で書くようにした  
(一応何行書いてもそれだけユーザーが作成されるようにしてあるはず)
```
ユーザー名=パスワード
```

## その他
動かす際にはローカルでマウントするところのディレクトリに `public` と `ユーザー名` を作成しておく必要がある  
今回は元々データがあるhddを割り当てたかったのでディレクトリがすでにある体で作った  
新しいものを割り当てるときはもう少しその辺り自動化しても良いかも
