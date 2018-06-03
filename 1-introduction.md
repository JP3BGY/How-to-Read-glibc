# 読み始める前に

## このドキュメントの対象者

このドキュメントではC言語の基本文法の理解し、gccやgdb、make(not Makefile)等のUNIX系の基礎的な開発環境の使い方を知っている人を対象としている。もし、わからない点やこのドキュメントの改善すべき点があればissueにて対応する。

## デバッガとデバッグシンボルのインストール
Ubuntu
```
sudo apt install libc6-dbg glibc-source gdb
```
Fedora
```
sudo dnf install gdb
sudo dnf debuginfo-install glibc
```

## GDB FrontEndについて

ソースナビゲーションを使わずにGDBとデバッグシンボルのみでソースを読むことも可能である。その場合大抵のパッケージマネージャから入れるデバッグシンボルは```-g2```でコンパイルするのでプリプロセッサの一部の情報がなくなっている事があることに注意しよう。また、GDBのTUIを用いるのも悪くはないが外部のGDB FrontEndの方が視覚から入る情報量が多いため格段に読みやすくなる。なので、ここで少しおすすめのGDB FrontEndを紹介する。なお、もっと知りたい場合は[GDB Wiki](https://sourceware.org/gdb/wiki/GDB%20Front%20Ends)を参照してほしい。

* [Eclipse CDT Standalone Debugger](https://wiki.eclipse.org/CDT/StandaloneDebugger)

おそらく一番機能が充実していて、一番使いやすいであろうもの。Reverse Debugにも対応しているのでこだわりがなければこれを使うのが一番だと思われる。

* cgdb

vim likeなキーバインドのCUI FrontEnd。CUI&vimにこだわりがあるならこれがベストであろう。各自使用しているパッケージマネージャからインストールできるはずである。

## ソースコードナビゲーション

ただgrepとエディタを用いて読むのでは日が暮れてしまうのでここではソースコードナビゲーションシステムを用いてソースリーディングを行う。ここではwoboqかSourceWebかDXRを使っている前提で話を進めるがある程度自信とこだわりがあれば他のものを使っていても問題なく読み進められるだろう。

* [woboq codebrowser](https://github.com/woboq/woboq_codebrowser/)

Clangを用いたコードナビゲーションシステム。プリプロセッサもある程度補足してくれるらしく関数をすぐに見つけることができる。
[サーバー](https://code.woboq.org/)も公開しており一般的なOSSのリーディングを用意無しですぐにすることが可能。いろいろ用意が面倒な人はこれを使うのがいいと思われる。たまにリファレンスが効かないことがあるがGoogle検索を用いることができるので大抵は困ることはない。

* [SourceWeb](https://github.com/rprichard/sourceweb)

C/C++にしか対応していないがコンパイルの手順を一通り追うことでプリプロセッサ等の処理が可能であり、使わない部分と使う部分が明確にわかる・精度が非常に高い等多くの利点がある。・・・のだが、どうやらFedoraではうまくビルドが動かないようである。原因は[このあたり](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=70936)で```-isystem```オプションが悪さしてstdlib.hを見えなくしているというものらしい。もしUbuntu等で動くのならばそれで良いのだが動かなければ早々に諦めたほうが良さそう。ちなみに私は```./build/clang-indexer```のMakefileの```-isystem /usr/include```を```-I/usr/include```に無理やり書き換えてビルドを通した。一応無事に動いてはいる。こちらもたまにリファレンスが効かないことがあるが、全体検索機能はないのでgrep等を用いることを推奨する。

* [ctags](https://github.com/universal-ctags/ctags), [cscope](http://cscope.sourceforge.net/), [GNU global](https://www.gnu.org/software/global/)

よく使われるタグシステム。大規模なソースでも比較的高速に動作する。また、エディタとの親和性も高く読みながら変更を加えるのには便利。
ただ、プリプロセッサをパースしたりはしないのでプリプロセッサ経由で宣言される変数や関数を検知できない。
ctagsとGNU globalはプラグインを使えば対応言語も多いので多くの言語が混在する場合には便利かもしれない。
今回はプリプロセッサの使用が多く対応できないので使わないこととする。

* [DXR](https://github.com/mozilla/dxr)

Mozillaが開発しているコードナビゲーションシステム。Firefox等に使うために開発されているようで、C/C++以外にJavascriptやRustなどにも対応している。デザインには難ありなところがあるがリンク等は精度が高いので非常に便利である。

* IDE

IDEも一種のナビゲーションシステムとしてソースリーディングに用いることは可能である。EclipseやVisual StudioでMakefile projectを読み込ませれば良いのでさほど難しいことではない。IDEの全機能をリーディングに用いることができるのでそれなりに快適に読める。ただ、非常に読み込みが遅く動作ももっさりするので勧めることはしない。

* [LXR](http://sourceforge.net/projects/lxr)
* [OpenGrok](http://oracle.github.io/opengrok/)
* [Gonzui](http://gonzui.sourceforge.net/)
* [elixir](https://github.com/free-electrons/elixir)
