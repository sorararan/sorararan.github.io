# go言語環境構築
goの環境構築をしたときのメモ

## goenvを使う
```
git clone https://github.com/syndbg/goenv.git ~/.goenv
```

## .zshrc
`.goenv` にgoenvを、 `.go` にパッケージなどを入れるように設定  
goenvが勝手に `GOPATH` を設定しちゃうので、 `GOENV_DISABLE_GOPATH` で抑制
```
export GOENV_ROOT=$HOME/.goenv
export PATH=$GOENV_ROOT/bin:$PATH
export GOPATH=$HOME/.go
export GOENV_DISABLE_GOPATH=1
eval "$(goenv init -)"
```

## コマンド
localは適宜変更
```
goenv install -l
goenv install バージョン名
goenv global バージョン名
mkdir .go
source .zshrc
```
