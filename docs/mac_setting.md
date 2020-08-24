# mac開発環境構築
mac開発環境構築について、これは使うだろうってものをメモしておく  
(なんか新しいの見つけたり古くなったら適宜変わるはず)

## Homebrew
パッケージマネージャーとしてHomebrewを使う
```
xcode-select --install
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## アプリケーションのインストール
```
brew cask install karabiner-elements
brew cask install google-japanese-ime
brew cask install iterm2
brew cask install google-chrome
brew cask install slack
brew cask install visual-studio-code
```
