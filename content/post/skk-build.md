---
title: "[archlinux] libskkとibus-skkをビルドする"
date: 2018-07-21T22:59:54+09:00
draft: false
---
# 背景  
Archlinuxの公式パッケージはlibskk 1.0.3-1で最新ではないから。  
何故最新のやつが使いたいのか→Mastodonの検索フォームが使えない、Google検索のアレがアレで辛い  

# ビルドする  
まずインストール済のlibskkをアンインストールする。(ibus-skkなどを使っている場合にはそれも消す)  
```
pacman -R libskk  
```   

GitHubからlibskkとibus-skkを持ってくる  
```  
git clone https://github.com/ueno/libskk
git clone https://github.com/ueno/ibus-skk  
```  

libskkをビルド  
```  
./autogen.sh --prefix=/usr  
make  
make install  
```  

ibus-skkもビルド  
```
./autogen.sh  
make  
make install  
```  

おわり  




