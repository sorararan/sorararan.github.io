# dnsmasq設定
家でサーバーを使う際に、手軽に名前解決できるようにdnsmasqを立てたときのメモ

## 構成
環境をあまり汚さないようにdockerでやることにした
```
dns:
  - Dockerfile
  - docker-compose.yml
  dnsmasq:
    - dnsmasq.conf
    - dnsmasq_resolv.conf
    - hosts_dnsmasq
```

## docker関連
### `Dockerfile`
ラズパイで動かす際にarmプロセッサ用の `dnsmasq` イメージがあるのか知らんので `alpine` からイメージを作った
```
FROM alpine:latest

RUN apk update \
  && apk add --no-cache dnsmasq

RUN mv /etc/dnsmasq.conf /etc/dnsmasq.conf.default

COPY ./dnsmasq/dnsmasq.conf /etc/dnsmasq.conf
COPY ./dnsmasq/dnsmasq_resolv.conf /etc/dnsmasq_resolv.conf
COPY ./dnsmasq/hosts_dnsmasq /etc/hosts_dnsmasq

EXPOSE 53/tcp 53/udp
CMD ["dnsmasq", "-k"]
```

### `docker-compose.yml`
`様々なネットワーク関連処理の実施` できるようにする `NET_ADMIN` が必要らしい
```
version: "3"
services:
  dnsmasq:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "53:53/udp"
      - "53:53/tcp"
    cap_add:
      - NET_ADMIN
    restart: always
```

## dnsmasq関連
`dnsmasq_resolv.conf` と `hosts_dnsmasq` は `/etc/resolv.conf` と `/ets/hosts` と同じ用に設定したいものを書けばいい

### `dnsmasq.conf`
```
user=root

port=53

# "."またはドメインを含まない名前は上位dnsにフォワードしない
domain-needed
# プライベートipは上位dnsにフォワードしない
bogus-priv

# ローカルドメイン設定
local=/名前/
# ドメイン自動補正(下のdomainで指定された名前を補完)
expand-hosts
domain=名前

# 自分で書いたhostsファイルを設定
no-hosts
addn-hosts=/etc/hosts_dnsmasq

# 自分で書いたresolv.confファイルを指定
resolv-file=/etc/dnsmasq_resolv.conf
```

## その他
ルータのdns設定も忘れずに  
うちのルータでは設定自体ができず、各マシンで設定した(もっと良いのほしいorz)
