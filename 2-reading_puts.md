# puts()を読んでみよう

## puts()を見つけよう

puts()を読み始めるためにはまずputs()がどこで宣言されているか見つけなくてはならない。そのための方法をいくつか紹介する。

* デバッグシンボルから場所を特定する

[読み始める前に](1-introduction.md)でインストールしたデバッグシンボルを用いてソースがどこにあるのかを特定する。まず、以下のようなプログラムを書きコンパイルをしよう。ここではプログラム本体のデバッグを目的とはしていないので```-g```オプションを付ける必要はない。
<details>
  <summary>Sample Program</summary>
  
```C
#include <stdio.h>
int main(){
  puts("Hello!!");
}
```

</details>
その後、GDBでプログラムを起動し、以下のようなコマンドを実行する。
<details>
  <summary>Sample gdb commands</summary>
  
```gdb
tui enable
b puts
r
```

</details>
そうすると、puts()にはいったところでプログラムが止まるのでそのファイル名を確認してソースコードに対して```find ./ -name filename.c```などで検索をかけたり、そのままGDBで読み進めれば良いと思う。
デバッガを使う利点は関数がシステムや設定によって宣言場所が違う場合に、自分の環境ではどれが使われているのかがすぐに分かるという点である。また、デバッグシンボルが使えるものならばすべてのソフトウェアで対応できる手法なので最初はこれで見つけ出すのが一番早いであろう。
難点としてはデバッグシンボルがない、または何らかの理由でデバッグすることができない場合は使えないというところであろう。しかし、今現在UbuntuとFedoraではパッケージマネージャから多くのデバッグシンボルをインストールできるので、その範囲内のソフトウェアなら特に問題とはならない。

* ナビゲーションシステムから探す。

各ナビゲーションシステムにあるシンボル検索や関数検索から検索する。ここではwoboqとSourceWebで動作を確認したがどちらも正しいところを示している。
こちらの利点はソースがあれば問題なくできるところ、いちいちソフトウェアを起動する必要がないのでMySQLやLinuxといった複雑なソフトウェアでも問題なく読めることであろう。欠点はそこまで結果に対して信用できないことである。一応WoboqやSourceWebやDXRはC/C++の場合はコンパイラを通しているらしいがgccではないので一部のgcc拡張には対応できないし、いくつかシンボルを検知できていないものも散見される。最終的にはやはりgrepを使うことになる。

* ソースコードの構造から当たりをつけて見つける

これはある程度ソースコードを読み慣れないとできないことだが、慣れればこれが一番良いと思われる。なぜなら、今回は標準ライブラリを読むという点からたまたま読みたい関数を指定することができたが一般にソフトウェアのリーディングをする場合はどの関数を読めばいいかなんてものは知ることができないからである。せいぜい特定の機能に関する部分というところまでしか絞ることができない。その場合は当たりをつけるということが非常に重要になる。ここではそのあたりの付け方について記す。

まず、glibcのトップのファイル構造を見てみよう。以下のリンクか自身がダウンロードしたディレクトリで```ls```を実行してみよう。以下のような結果になる。
https://code.woboq.org/userspace/glibc/
<details>
  <summary>ls glibc</summary>
  
```
COPYING        NEWS         config.make.in  extra-lib.mk    inet               malloc         po              socket        test-skeleton.c
COPYING.LIB    README       configure       gen-locales.mk  intl               manual         posix           soft-fp       time
ChangeLog      Rules        configure.ac    gmon            io                 math           pwd             stdio-common  timezone
ChangeLog.old  abi-tags     conform         gnulib          libc-abis          mathvec        resolv          stdlib        version.h
INSTALL        aclocal.m4   crypt           grp             libidn             misc           resource        streams       wcsmbs
LICENSES       argp         csu             gshadow         libio              nis            rt              string        wctype
MAINTAINERS    assert       ctype           hesiod          libof-iterator.mk  nptl           scripts         sunrpc
Makeconfig     benchtests   debug           hurd            locale             nptl_db        setjmp          support
Makefile       bits         dirent          iconv           localedata         nscd           shadow          sysdeps
Makefile.in    catgets      dlfcn           iconvdata       login              nss            shlib-versions  sysvipc
Makerules      config.h.in  elf             include         mach               o-iterator.mk  signal          termios
```

</details>
ここで注目してほしいのがファイルやディレクトリの名前である。見て分かる通りこれらの名前は変数名等と同じように意味を持つものになっている。
例えば"assert"はC言語の標準ヘッダである"assert.h"関連の関数があるであろうと予測できるし、"time"も同様"time.h"関連であろうということが予測できる。
また、わからないものは調べれば大抵はすぐに出てくる。例えば"elf"や"hurd"などは有名なのでこれらは問題なく出てくる。
これで大まかにどこに何があるのかの見当がついた。

それではここでputs()がどこにあるのかを予想してみよう。まず、puts()がおいてあるヘッダは"stdio.h"である。この名前から怪しそうなのは"stdio-common"と"libio"と"io"であろう。だが、現時点ではこれらの明確な差はわかっていないのでとりあえずそれぞれ何がはいっているのか```ls io libio stdio-common```で見てみよう。
<details>
  <summary>ls io libio stdio-common</summary>
  
```
io:
Makefile                  fchdir.c      ftwtest-sh    lxstat64.c         posix_fallocate64.c  tst-copy_file_range-compat.c  tst-renameat.c
Versions                  fchmod.c      ftwtest.c     mkdir.c            ppoll.c              tst-copy_file_range.c         tst-statvfs.c
access.c                  fchmodat.c    futimens.c    mkdirat.c          pwd.c                tst-faccessat.c               tst-symlinkat.c
bits                      fchown.c      fxstat.c      mkfifo.c           read.c               tst-fchmodat.c                tst-ttyname_r.c
bug-ftw1.c                fchownat.c    fxstat64.c    mkfifoat.c         readlink.c           tst-fchownat.c                tst-unlinkat.c
bug-ftw2.c                fcntl.c       fxstatat.c    mknod.c            readlinkat.c         tst-fcntl.c                   ttyname.c
bug-ftw3.c                fcntl.h       fxstatat64.c  mknodat.c          rmdir.c              tst-fstatat.c                 ttyname_r.c
bug-ftw4.c                flock.c       getcwd.c      open.c             sendfile.c           tst-fts-lfs.c                 umask.c
bug-ftw5.c                fstat.c       getdirname.c  open64.c           sendfile64.c         tst-fts.c                     unlink.c
chdir.c                   fstat64.c     getwd.c       open64_2.c         stat.c               tst-futimesat.c               unlinkat.c
chmod.c                   fstatat.c     isatty.c      open_2.c           stat64.c             tst-getcwd-abspath.c          utime.c
chown.c                   fstatat64.c   lchmod.c      openat.c           statfs.c             tst-getcwd.c                  utime.h
close.c                   fstatfs.c     lchown.c      openat64.c         statfs64.c           tst-linkat.c                  utimensat.c
copy_file_range-compat.c  fstatfs64.c   link.c        openat64_2.c       statvfs.c            tst-mkdirat.c                 write.c
copy_file_range.c         fstatvfs.c    linkat.c      openat_2.c         statvfs64.c          tst-mkfifoat.c                xmknod.c
creat.c                   fstatvfs64.c  lockf.c       pipe.c             symlink.c            tst-mknodat.c                 xmknodat.c
creat64.c                 fts.c         lockf64.c     pipe2.c            symlinkat.c          tst-open-tmpfile.c            xstat.c
dup.c                     fts.h         lseek.c       poll.c             sys                  tst-openat.c                  xstat64.c
dup2.c                    fts64.c       lseek64.c     poll.h             test-lfs.c           tst-posix_fallocate-common.c
dup3.c                    ftw.c         lstat.c       posix_fadvise.c    test-stat.c          tst-posix_fallocate.c
euidaccess.c              ftw.h         lstat64.c     posix_fadvise64.c  test-stat2.c         tst-posix_fallocate64.c
faccessat.c               ftw64.c       lxstat.c      posix_fallocate.c  test-utime.c         tst-readlinkat.c

libio:
Depend             clearerr_u.c   getwc_u.c      iogets.c          oldiofsetpos64.c  tst-eof.c                   tst-widetext.c
Makefile           fcloseall.c    getwchar.c     iogetwline.c      oldiopopen.c      tst-ext.c                   tst-widetext.input
Versions           feof.c         getwchar_u.c   iolibio.h         oldpclose.c       tst-ext2.c                  tst-wmemstream1.c
__fbufsize.c       feof_u.c       iofclose.c     iopadn.c          oldstdfiles.c     tst-fgetc-after-eof.c       tst-wmemstream2.c
__flbf.c           ferror.c       iofdopen.c     iopopen.c         oldtmpfile.c      tst-fgetwc.c                tst-wmemstream3.c
__fpending.c       ferror_u.c     iofflush.c     ioputs.c          pclose.c          tst-fgetwc.input            tst_getwc.c
__fpurge.c         filedoalloc.c  iofflush_u.c   ioseekoff.c       peekc.c           tst-fgetws.c                tst_getwc.input
__freadable.c      fileno.c       iofgetpos.c    ioseekpos.c       putc.c            tst-fopenloc.c              tst_putwc.c
__freading.c       fileops.c      iofgetpos64.c  iosetbuffer.c     putc_u.c          tst-fopenloc2.c             tst_swprintf.c
__fsetlocking.c    fmemopen.c     iofgets.c      iosetvbuf.c       putchar.c         tst-fputws.c                tst_swscanf.c
__fwritable.c      fputc.c        iofgets_u.c    ioungetc.c        putchar_u.c       tst-freopen.c               tst_wprintf.c
__fwriting.c       fputc_u.c      iofgetws.c     ioungetwc.c       putwc.c           tst-fseek.c                 tst_wprintf2.c
bits               fputwc.c       iofgetws_u.c   iovdprintf.c      putwc_u.c         tst-ftell-active-handler.c  tst_wscanf.c
bug-fopena+.c      fputwc_u.c     iofopen.c      iovsprintf.c      putwchar.c        tst-ftell-append.c          tst_wscanf.input
bug-fseek.c        freopen.c      iofopen64.c    iovsscanf.c       putwchar_u.c      tst-ftell-partial-wide.c    vasprintf.c
bug-ftell.c        freopen64.c    iofopncook.c   iovswscanf.c      rewind.c          tst-fwrite-error.c          vscanf.c
bug-memstream1.c   fseek.c        iofputs.c      iowpadn.c         setbuf.c          tst-memstream1.c            vsnprintf.c
bug-mmap-fflush.c  fseeko.c       iofputs_u.c    libc_fatal.c      setlinebuf.c      tst-memstream2.c            vswprintf.c
bug-rewind.c       fseeko64.c     iofputws.c     libio.h           stdfiles.c        tst-memstream3.c            vtables.c
bug-rewind2.c      ftello.c       iofputws_u.c   libioP.h          stdio.c           tst-mmap-eofsync.c          vwprintf.c
bug-ungetc.c       ftello64.c     iofread.c      memstream.c       stdio.h           tst-mmap-fflushsync.c       vwscanf.c
bug-ungetc2.c      fwide.c        iofread_u.c    obprintf.c        strfile.h         tst-mmap-offend.c           wfiledoalloc.c
bug-ungetc3.c      fwprintf.c     iofsetpos.c    oldfileops.c      strops.c          tst-mmap-setvbuf.c          wfileops.c
bug-ungetc4.c      fwscanf.c      iofsetpos64.c  oldfmemopen.c     swprintf.c        tst-mmap2-eofsync.c         wgenops.c
bug-ungetwc1.c     genops.c       ioftell.c      oldiofclose.c     swscanf.c         tst-popen1.c                wmemstream.c
bug-ungetwc2.c     getc.c         iofwide.c      oldiofdopen.c     test-fmemopen.c   tst-setvbuf1.c              wprintf.c
bug-wfflush.c      getc_u.c       iofwrite.c     oldiofgetpos.c    test-freopen.c    tst-sscanf.c                wscanf.c
bug-wmemstream1.c  getchar.c      iofwrite_u.c   oldiofgetpos64.c  test-freopen.sh   tst-swscanf.c               wstrops.c
bug-wsetpos.c      getchar_u.c    iogetdelim.c   oldiofopen.c      tst-atime.c       tst-ungetwc1.c
clearerr.c         getwc.c        iogetline.c    oldiofsetpos.c    tst-bz22415.c     tst-ungetwc2.c

stdio-common:
Depend                bug22.c         fxprintf.c         psignal.c       siglist.c         tst-fileno.c           tst-sprintf.c
Makefile              bug23-2.c       gentempfd.c        putw.c          snprintf.c        tst-fmemopen.c         tst-sprintf2.c
Versions              bug23-3.c       getline.c          reg-modifier.c  sprintf.c         tst-fmemopen2.c        tst-sprintf3.c
_i18n_number.h        bug23-4.c       getw.c             reg-printf.c    sscanf.c          tst-fmemopen3.c        tst-sscanf.c
_itoa.c               bug23.c         isoc99_fscanf.c    reg-type.c      stdio_ext.h       tst-fmemopen4.c        tst-swprintf.c
_itowa.c              bug24.c         isoc99_scanf.c     remove.c        stdio_lim.h.in    tst-fphex-wide.c       tst-swscanf.c
_itowa.h              bug25.c         isoc99_sscanf.c    rename.c        tempnam.c         tst-fphex.c            tst-tmpnam.c
asprintf.c            bug26.c         isoc99_vfscanf.c   renameat.c      tempname.c        tst-fseek.c            tst-unbputc.c
bits                  bug3.c          isoc99_vscanf.c    scanf.c         temptest.c        tst-fwrite.c           tst-unbputc.sh
bug-vfprintf-nargs.c  bug4.c          isoc99_vsscanf.c   scanf1.c        test-fseek.c      tst-gets.c             tst-ungetc.c
bug1.c                bug5.c          itoa-digits.c      scanf10.c       test-fwrite.c     tst-gets.input         tst-unlockedio.c
bug1.input            bug6.c          itoa-udigits.c     scanf11.c       test-popen.c      tst-grouping.c         tst-vfprintf-mbs-prec.c
bug10.c               bug6.input      itowa-digits.c     scanf12.c       test-vfprintf.c   tst-long-dbl-fphex.c   tst-vfprintf-user-type.c
bug11.c               bug7.c          perror.c           scanf12.input   test_rdwr.c       tst-obprintf.c         tst-vfprintf-width-prec.c
bug12.c               bug8.c          printf-parse.h     scanf13.c       tfformat.c        tst-perror.c           tst-wc-printf.c
bug13.c               bug9.c          printf-parsemb.c   scanf14.c       tiformat.c        tst-popen.c            tstdiomisc.c
bug14.c               ctermid.c       printf-parsewc.c   scanf15.c       tllformat.c       tst-popen2.c           tstgetln.c
bug16.c               cuserid.c       printf-prs.c       scanf16.c       tmpfile.c         tst-printf-bz18872.sh  tstgetln.input
bug17.c               dprintf.c       printf.c           scanf17.c       tmpfile64.c       tst-printf-round.c     tstscanf.c
bug18.c               errlist.c       printf.h           scanf2.c        tmpnam.c          tst-printf.c           tstscanf.input
bug18a.c              errnobug.c      printf_fp.c        scanf3.c        tmpnam_r.c        tst-printf.sh          vfprintf.c
bug19.c               flockfile.c     printf_fphex.c     scanf4.c        tst-cookie.c      tst-printfsz.c         vfscanf.c
bug19a.c              fprintf.c       printf_size.c      scanf5.c        tst-fdopen.c      tst-put-error.c        vfwprintf.c
bug2.c                fscanf.c        psiginfo-data.h    scanf7.c        tst-ferror.c      tst-rndseek.c          vfwscanf.c
bug20.c               ftrylockfile.c  psiginfo-define.h  scanf8.c        tst-ferror.input  tst-setvbuf1.c         vprintf.c
bug21.c               funlockfile.c   psiginfo.c         scanf9.c        tst-fgets.c       tst-setvbuf1.expect    xbug.c
```

</details>
非常に多くのファイルが有って少し驚くかもしれないが大したことはない。よく見てみると標準関数の名前が並んでいることがわかる。```test-*```や```tst-*```はおそらくテストケースが入っているファイルであろう。ここまで来たらどこにあるかはすぐに分かると思われる。そう、```libio/ioputs.c```である。

このようによく管理されているプロジェクトは楽に目的の部分にたどり着くことができるようにちゃんと作られているのだ。他のプロジェクトの場合も同様である。はじめは見てすぐにこのようなことを理解することは難しいかもしれないがめげずに経験を積むようにしてほしい。

## puts()を読んでみよう。
ようやくここでputs()の関数を読むことにしよう。各自ナビゲーションシステムやデバッガでputs()を表示してほしいが一応ioputs.cの全文をここにも貼ることにする。
<details>
  <summary>glibc/libio/ioputs.c</summary>
  
```C
/* Copyright (C) 1993-2018 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.

   The GNU C Library is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
   Lesser General Public License for more details.

   You should have received a copy of the GNU Lesser General Public
   License along with the GNU C Library; if not, see
   <http://www.gnu.org/licenses/>.

   As a special exception, if you link the code in this file with
   files compiled with a GNU compiler to produce an executable,
   that does not cause the resulting executable to be covered by
   the GNU Lesser General Public License.  This exception does not
   however invalidate any other reasons why the executable file
   might be covered by the GNU Lesser General Public License.
   This exception applies to code released by its copyright holders
   in files containing the exception.  */

#include "libioP.h"
#include <string.h>
#include <limits.h>

int
_IO_puts (const char *str)
{
  int result = EOF;
  size_t len = strlen (str);
  _IO_acquire_lock (_IO_stdout);

  if ((_IO_vtable_offset (_IO_stdout) != 0
       || _IO_fwide (_IO_stdout, -1) == -1)
      && _IO_sputn (_IO_stdout, str, len) == len
      && _IO_putc_unlocked ('\n', _IO_stdout) != EOF)
    result = MIN (INT_MAX, len + 1);

  _IO_release_lock (_IO_stdout);
  return result;
}

weak_alias (_IO_puts, puts)
```

</details>
まずはじめに気になるのが```weak_alias (_IO_puts, puts)```であろう。そこでweak_aliasの定義を確認してみよう。
<details>
  <summary>glibc/include/libc-symbols.h</summary>
  
```C
# define weak_alias(name, aliasname) _weak_alias (name, aliasname)
# define _weak_alias(name, aliasname) \
  extern __typeof (name) aliasname __attribute__ ((weak, alias (#name)));
```

</details>
いきなり訳がわからないとは思うのでここは少しわかりやすく分け目を大きくしてみようと思う。
<details>
  <summary>glibc/include/libc-symbols.h modified</summary>
  
```C
# define weak_alias(name, aliasname) _weak_alias (name, aliasname)
# define _weak_alias(name, aliasname) \
  extern       __typeof (name)         aliasname         __attribute__ ((weak, alias (#name)));
```
</details>
これで少しはわかりやすくなったかと思う。
ひとつずつ見ていこう、まずexternは外部に宣言がされているという意味の修飾子だ。これは、関数の場合は意味をなさないが変数の場合は存在のみを定義して実態を作らないという意味を持つ。が、ここでは```aliasname```は```puts```、即ち関数なのでとりあえず無視しよう。
次に```__typeof (name)```である。これは[typeof](https://gcc.gnu.org/onlinedocs/gcc/Typeof.html)演算子であろうというのは簡単に予測できる。
最後が```__attribute__ ((weak, alias (#name)));```である。これは"alias gcc extension"等で検索すれば出てくるgccの拡張機能である[alias](https://gcc.gnu.org/onlinedocs/gcc-4.7.2/gcc/Function-Attributes.html)だ。簡単に言えば別名というわけだ。weakとあるのは同ページの下の方に記述があるが別で宣言があったら上書きできるという意味だ。つまりputs()という名前の別の関数をプログラムで定義しても問題なくコンパイルされ、かつputs()の呼び出し時はその新しく書いた方が使われるようになるわけだ。C言語では普通名前が衝突するとエラーが出るので普通の挙動ではないことがわかるだろう。``#name``の部分はその部分をそのまま文字列、即ち```"name"```にするという意味だ。こうする理由は上のリンクをよく見ればわかるが__attribute__に使う引数が文字列である必要があるからだ。
結果として```weak_alias(_IO_puts,puts)```は以下のように展開される。
```C
extern __typeof (_IO_puts) puts __attribute__ ((weak, alias ("_IO_puts")));
```
こうして、putsは_IO_putsのaliasであると定義されるわけだ。

putsの謎が解けたところでその中身を読んでいこう。
<details>
  <summary>glibc/libio/ioputs.c</summmary>
  
```C
int
_IO_puts (const char *str)
{
  int result = EOF;
  size_t len = strlen (str);
  _IO_acquire_lock (_IO_stdout); // 1

  if ((_IO_vtable_offset (_IO_stdout) != 0 // 2
       || _IO_fwide (_IO_stdout, -1) == -1) // 3
      && _IO_sputn (_IO_stdout, str, len) == len // 4
      && _IO_putc_unlocked ('\n', _IO_stdout) != EOF) // 4
    result = MIN (INT_MAX, len + 1);

  _IO_release_lock (_IO_stdout); // 1
  return result;
}
```
</details>
読みやすさのために番号を振った。

一番最初につまずくのは1番の```_IO_acquire_lock``` ```_IO_release_lock```だと思う。これはマルチスレッドプログラミングで使われる[lock free](https://ja.wikipedia.org/wiki/Lock-free%E3%81%A8Wait-free%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0)と呼ばれるものだ。かいつまんで話すとマルチスレッドプログラミングをするときに同じリソースに対して違うスレッドが同時に使うという状況が起こることがある。それを単純に許してしまうと同時に別の値が上書きされたりして、結果挙動がおかしくなることがある。それを防ぐためにあるスレッドが使っている限りは別のスレッドからは使えないように鍵をかけ、使い終わったらそれを開放する、という意味でロックフリーというわけだ。この部分はこれ以上深くは追わないこととする。理由はwoboq等でマクロの展開結果を見ればわかるがとんでもない量のコードになっているからだ。ここはputs()の核心ではないのであまり時間をかけないためにとばすことにする。

次に2番の部分だが、結論からするとこの部分では何もしない、何もおこらないことに普通はなっている。
どういうことか定義部分を見てみよう。
<details>
  <summary>glibc/libio/libioP.h</summary>
  
```C
/* Setting this macro to 1 enables the use of the _vtable_offset bias
   in _IO_JUMPS_FUNCS, below.  This is only needed for new-format
   _IO_FILE in libc that must support old binaries (see oldfileops.c).  */
#if SHLIB_COMPAT (libc, GLIBC_2_0, GLIBC_2_1) && !defined _IO_USE_OLD_IO_FILE
# define _IO_JUMPS_OFFSET 1
#else
# define _IO_JUMPS_OFFSET 0
#endif

// CUT

#if _IO_JUMPS_OFFSET
# define _IO_JUMPS_FUNC(THIS) \
  (IO_validate_vtable                                                   \
   (*(struct _IO_jump_t **) ((void *) &_IO_JUMPS_FILE_plus (THIS)        \
                             + (THIS)->_vtable_offset)))
# define _IO_vtable_offset(THIS) (THIS)->_vtable_offset
#else
# define _IO_JUMPS_FUNC(THIS) (IO_validate_vtable (_IO_JUMPS_FILE_plus (THIS)))
# define _IO_vtable_offset(THIS) 0
#endif
```
</details>
注目してほしいのが```_IO_JUMPS_OFFSET```の定義をしている上の部分のコメントである。単純に言えばold binary用だから今は使ってないということだ。
ここでは具体的な言及は避けるが、実行ファイルにも新旧や種類や変更があっておそらくこれは昔の実行ファイルを使う場合に必要になるものなのだと推測できる。
実際GDBでステップ実行をすると＿IO_vtable_offsetの部分はコンパイル時に評価がfalseであると予め計算されるため実行ファイル上ではなかったもの扱いになっている。

次に3番だが、fwideについては[man page](https://linuxjm.osdn.jp/html/LDP_man-pages/man3/fwide.3.html)が存在しておりおそらく同じ関数だと予測できる。ソースを確認してもよいがソースを読むときに重要なのは必要のないところは読まないことなのでここではこれ以上の言及はしない。この部分は要はstdoutをバイトストリームに変更するだけである。

4番についてだが、デバッガで確認すればわかるがここが出力を行っている関数である。そのため、まずここを読む前に_IO_stdoutの型を調べようと思う。ソースを読む上ではロジックを読むのも重要であるがそれ以上に型、特にstruct等ユーザーが定義した型の情報が大事になる。なぜなら、処理というのは型の意味する情報があって初めて作られるからだ。では、早速見てみよう。
<details>
  <summary>glibc/libio/bits/types/struct_FILE.h</summary>
  
```C
struct _IO_FILE
{
  int _flags;                /* High-order word is _IO_MAGIC; rest is flags. */
  /* The following pointers correspond to the C++ streambuf protocol. */
  char *_IO_read_ptr;        /* Current read pointer */
  char *_IO_read_end;        /* End of get area. */
  char *_IO_read_base;        /* Start of putback+get area. */
  char *_IO_write_base;        /* Start of put area. */
  char *_IO_write_ptr;        /* Current put pointer. */
  char *_IO_write_end;        /* End of put area. */
  char *_IO_buf_base;        /* Start of reserve area. */
  char *_IO_buf_end;        /* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */
  struct _IO_marker *_markers;
  struct _IO_FILE *_chain;
  int _fileno;
  int _flags2;
  __off_t _old_offset; /* This used to be _offset but it's too small.  */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];
  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
struct _IO_FILE_complete
{
  struct _IO_FILE _file;
#endif
  __off64_t _offset;
  /* Wide character stream stuff.  */
  struct _IO_codecvt *_codecvt;
  struct _IO_wide_data *_wide_data;
  struct _IO_FILE *_freeres_list;
  void *_freeres_buf;
  size_t __pad5;
  int _mode;
  /* Make sure we don't get into trouble again.  */
  char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];
};
```
</details>
これまた長くて大変だが、すべてを理解する必要はない。とりあえずは名前から推測してみよう。その前にあまり知らないであろう単語の説明をする。
* offset

直訳すると「埋め合わせ・差引」だが、とりわけコンピュータ系では配列や構造体で始めから何バイト後ろにあるか、という意味で使われることが多い。

* vtable

省略された部分を書くと"vector table"で、コンピュータ系では関数ポインタの列(構造体)という意味になる。

* list 

アルゴリズムで使われる言葉で次のデータを指し示すポインタによって連結されたデータ構造のことである。chainもおそらく同じ意味で使われていると思われる。

* buf

今回の場合はおそらくlistと対照して普通のC言語の配列を意味するものだと思われる。

で、このデータ構造で気になるのが```_IO_USE_OLD_IO_FILE```だが基本OLDと書かれているものは互換性のために残ってるだけで今のPCで使われることはないと推測して良い。なのでこの部分はないものとして考える。ほかはとりあえずは今はコメントと単語から推測できる程度がわかっていれば問題ない。


では話を戻して4番を見ていこう。まず下のソースを見てほしい、これは4の関数っぽいものの宣言部分とそれに関連したところだ。
<details>
  <summary>glibc/libio/libioP.h</summary>

```c
#define _IO_WIDE_JUMPS_FUNC(THIS) _IO_WIDE_JUMPS(THIS)
#define JUMP_FIELD(TYPE, NAME) TYPE NAME
#define JUMP0(FUNC, THIS) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS)
#define JUMP1(FUNC, THIS, X1) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1)
#define JUMP2(FUNC, THIS, X1, X2) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1, X2)
#define JUMP3(FUNC, THIS, X1,X2,X3) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1,X2, X3)
#define JUMP_INIT(NAME, VALUE) VALUE
#define JUMP_INIT_DUMMY JUMP_INIT(dummy, 0), JUMP_INIT (dummy2, 0)

//CUT

/* The 'xsputn' hook writes upto N characters from buffer DATA.
   Returns EOF or the number of character actually written.
   It matches the streambuf::xsputn virtual function. */
typedef size_t (*_IO_xsputn_t) (FILE *FP, const void *DATA,
                                    size_t N);
#define _IO_XSPUTN(FP, DATA, N) JUMP2 (__xsputn, FP, DATA, N)

//CUT

struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
};
/* We always allocate an extra word following an _IO_FILE.
   This contains a pointer to the function jump table used.
   This is for compatibility with C++ streambuf; the word can
   be used to smash to a pointer to a virtual function table. */
struct _IO_FILE_plus
{
  FILE file;
  const struct _IO_jump_t *vtable;
};

//CUT

#define _IO_sputn(__fp, __s, __n) _IO_XSPUTN (__fp, __s, __n)
```

</details>
ここのあたりは少し技巧的である。何がしたくてこんな長ったらしくなっているかというとC++との互換性のためにC++のクラスと同じことを実現しようとしているのだ。

ここで少しC++のクラスの内部の挙動について解説すると、ほぼ上のコードと同じなのだが各クラスごとにベクターテーブルと呼ばれるメソッドのポインターをまとめた物がある。このベクターテーブルは継承関係にあるものは同じオフセットに同型同名(オーバーライドされていることもあるので同じメソッドとは限らない)のメソッドが存在して、これによってオーバーライドを実現している。
また、メソッドの呼び出し時にはベクターテーブルを参照して呼び出すわけだがメソッドの引数には隠し引数としてオブジェクトへのポインタが渡されている。このポインタを使ってオブジェクトのフィールドにアクセスするわけだ。

上のコードはつまりそれと同じ挙動をするプログラムを作成している。まずFILE構造体を拡張する形で_IO_FILE_plus構造体を作成している。stdin等をじっくり読んでいる人はすでに知っていると思うがstdin,out,errはすべて_IO_FILE_plus構造体で宣言されている。以下がその部分である。
<details>
  <summary>glibc/libio/stdfiles.c</summary>
  
```c
#ifdef _IO_MTSAFE_IO
# define DEF_STDFILE(NAME, FD, CHAIN, FLAGS) \
  static _IO_lock_t _IO_stdfile_##FD##_lock = _IO_lock_initializer; \
  static struct _IO_wide_data _IO_wide_data_##FD \
    = { ._wide_vtable = &_IO_wfile_jumps }; \
  struct _IO_FILE_plus NAME \
    = {FILEBUF_LITERAL(CHAIN, FLAGS, FD, &_IO_wide_data_##FD), \
       &_IO_file_jumps};
#else
# define DEF_STDFILE(NAME, FD, CHAIN, FLAGS) \
  static struct _IO_wide_data _IO_wide_data_##FD \
    = { ._wide_vtable = &_IO_wfile_jumps }; \
  struct _IO_FILE_plus NAME \
    = {FILEBUF_LITERAL(CHAIN, FLAGS, FD, &_IO_wide_data_##FD), \
       &_IO_file_jumps};
#endif

DEF_STDFILE(_IO_2_1_stdin_, 0, 0, _IO_NO_WRITES);
DEF_STDFILE(_IO_2_1_stdout_, 1, &_IO_2_1_stdin_, _IO_NO_READS);
DEF_STDFILE(_IO_2_1_stderr_, 2, &_IO_2_1_stdout_, _IO_NO_READS+_IO_UNBUFFERED);
```
</details>
非常にややこしいがこの部分は今は特別読み込む必要はないのでwoboqやgdbで展開結果だけを表示させれば良いと思う。とりあえず_IO_FILE_plus構造体で宣言されていることを知っていることが重要である。
