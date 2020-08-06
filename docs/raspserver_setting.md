# raspberry piサーバー構築
raspberry piを個人用サーバーとして使う際に、初めに設定したことをメモしておく

## osのインストール
### raspbian liteのイメージ
[ここ](https://www.raspberrypi.org/downloads/raspberry-pi-os/)からイメージを持ってくる

### sdカードのフォーマット
[sd card formatter](https://www.sdcard.org/jp/downloads/formatter/)が使える

### イメージの書き込み
[etcher](https://www.balena.io/etcher/)が使える


## ユーザー周りの設定
### rootユーザーのパスワード変更
デフォルトでrootのパスワードがないので、変更する
```
sudo passwd root
```

### piユーザーではなく、自分で設定したユーザーを使う
videoに入れておくとvcgencmd measure_tempでcpu温度が取れる、なくてもいい
```
sudo adduser ユーザー名
sudo gpasswd -a ユーザー名 sudo
sudo gpasswd -a ユーザー名 video
sudo gpasswd -d pi sudo
```

#### `/etc/sudoers.d/010_pi_nopasswd` の変更
```
# pi ALL=(ALL) NOPASSWD: ALL
```


## ホスト名の変更
`/etc/hosts` 、 `/etc/hostname` のraspberrypiを変更


## sshを使う
### デフォルト設定
```
sudo systemctl enable ssh
```

### 公開鍵を持ってくる
ホームディレクトリで
```
mkdir .ssh
```

鍵を持ってる方のマシンで
```
scp 公開鍵のパス ユーザー名@IPアドレス:~/.ssh/authorized_keys
```

### `/etc/ssh/sshd_config` の変更
```
Port 使うポート番号
PermitRootLogin no
PasswordAuthentication no
```


## ufwを使う
とりあえずsshだけ使うようにしておく(何かポート開放するときはufwの存在を忘れずに)
```
sudo apt-get install ufw
sudo ufw default deny
sudo ufw deny ssh
sudo ufw allow ポート番号
sudo ufw enable
```


## dockerを使う
```
curl -sSL https://get.docker.com | sh
sudo gpaswd -a ユーザー名 docker
sudo apt-get install docker-compose
```


## その他ソフトウェアのインストール
```
sudo apt-get install tmux vim
```

