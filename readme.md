# e-Stat-memo-ReView

e-Statのメモ  
執筆内容はこちらを参照  
[page_list.md](page_list.md)  

## 執筆環境構築

```
# Macの場合のTeXのインストール
brew update
brew install caskroom/cask/brew-cask
brew cask install mactex

# 要rubyインストール
bundle
```

## ビルド

```
# epubを出力するとき
rake epub
# pdfを出力するとき
rake pdf
```
