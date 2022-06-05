# SECCON Beginners CTF 2022
## はじめに
SECCON Beginners CTF 2022 [SECCON Beginners CTF 2022 を開催いたします！ - SECCON2022](https://www.seccon.jp/2022/seccon_beginners/content.html)に，playgroundというコミュニティの先輩方と3人で参加したので，僕が関わった問題についてWriteUPします。
CTFというものが完全に初めてだったので試行錯誤でしたが，意外と個人的には奮闘できたと思います。
## Web
### textex[128 team solved, 92pt]
>texをpdfにするサービスを作りました。
>texで攻撃することはできますか？
>
>https://textex.quals.beginners.seccon.jp
>textex.tar.gz 31fd2c7d44bf76b937e3d141032ac62b5e301dee

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.11.39.png)

Cloud LaTeXみたいな愉快なツールが登場した。

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.12.37.png)

ファイルはこんな感じで，`flag`を読み込めればよさそうだ。

TeXでは`\input`によりファイルの中身をそのまま（TeXのコードの一部として）読み込める。
```LaTeX
\documentclass{article}
\begin{document}

This is a sample.
\input{requirements.txt}

\end{document}
```
とすると，

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.17.34.png)

と読み込めた。

`flag`というファイルのテキストを引っ張り出せればOKだとわかったが，
```python
        # No flag !!!!
        if "flag" in tex_code.lower():
            tex_code = ""
```
というコードに阻まれていた。flagという文字列を作成できれば良いので，
```LaTeX
\newcommand{\aaa}{fl}
\newcommand{\bbb}{ag}
```
と適当に生成して，ファイル名を`\aaa\bbb`と指定したらうまくいった。

最後に，フラグに何らかのLaTeXの特殊文字が含まれていることがわかったので，そんな時はと`listings`を試したがインポートできなかった。調べてみて，`verbatim`をインポートすることに成功したので，最終的に
```LaTeX
\documentclass{article}
\usepackage{verbatim}
\newcommand{\aaa}{fl}
\newcommand{\bbb}{ag}
\begin{document}
\verbatiminput{\aaa\bbb}
\end{document}
```
でうまくいった。

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.23.10.png)

## misc
### phisher[238 team solved, 70pt]
>ホモグラフ攻撃を体験してみましょう。
>心配しないで！相手は人間ではありません。
>```bash
>nc phisher.quals.beginners.seccon.jp 44322
>```
>phisher.tar.gz bc81b6186868cb1d932a12bd5a3612010b52cb8d

入力したテキストを，`Murecho`というオープンなフォントで画像ファイルに書き出し，english指定でocrにかけ文字を復元していた。
このocrに掛け復元した文字が，`www.example.com`のどの文字も使わずに`www.example.com`と騙して読み込むことができればflagが帰ってくるようになっていた。

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.26.55.png)

~~実験レポで全角半角変換でお世話になっているサイトの~~ [Unicodeの一覧ツール](https://so-zou.jp/web-app/text/unicode/)をテキストエディタに貼り付け，Murechoフォントで表示した。その後はひたすら気合いで探しては試行錯誤を繰り返して，何とか`ωωω․εⅹáⅿρIε․ċοⅿ`という文字列でうまくひっかっかった。
ドットを探すのが特に大変だった。

```bash
❯ nc phisher.quals.beginners.seccon.jp 44322
       _     _     _                  ____    __
 _ __ | |__ (_)___| |__   ___ _ __   / /\ \  / /
| '_ \| '_ \| / __| '_ \ / _ \ '__| / /  \ \/ /
| |_) | | | | \__ \ | | |  __/ |    \ \  / /\ \
| .__/|_| |_|_|___/_| |_|\___|_|     \_\/_/  \_\
|_|

FQDN: ωωω․εⅹáⅿ𐌓Iε․ċοⅿ
"www.examile.com" is not "www.example.com" !!!!

❯ nc phisher.quals.beginners.seccon.jp 44322
       _     _     _                  ____    __
 _ __ | |__ (_)___| |__   ___ _ __   / /\ \  / /
| '_ \| '_ \| / __| '_ \ / _ \ '__| / /  \ \/ /
| |_) | | | | \__ \ | | |  __/ |    \ \  / /\ \
| .__/|_| |_|_|___/_| |_|\___|_|     \_\/_/  \_\
|_|

FQDN: ωωω․εⅹáⅿρIε․ċοⅿ
ctf4b{n16h7_ph15h1n6_15_600d}
```

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.00.23.png)

[文章比較ツール](https://lab.hidetake.org/diff/)より。

<img width="606" alt="Screen Shot 2022-06-05 at 19 14 54" src="https://user-images.githubusercontent.com/41906969/172045764-a4b696ee-03de-4518-bebc-dcbecf924967.png">

上: `www.example.com`
下: `ωωω․εⅹáⅿρIε․ċοⅿ`

みなさんも，ホモグラフ攻撃によるフィッシングに遭わないよう気をつけましょう（？）。

### hitchhike4b[125 team solved, 84pt]
>helpを呼び出したら、ページャーとして猫が来ました。
>```bash
>nc hitchhike4b.quals.beginners.seccon.jp 55433
>```
コードなしでの取得だった。先輩が`__main__`と入力するとフラグの前半がもらえることを突き止めてくれて，後半（`flag2`）を探した。

```
❯ nc hitchhike4b.quals.beginners.seccon.jp 55433
 _     _ _       _     _     _ _        _  _   _
| |__ (_) |_ ___| |__ | |__ (_) | _____| || | | |__
| '_ \| | __/ __| '_ \| '_ \| | |/ / _ \ || |_| '_ \
| | | | | || (__| | | | | | | |   <  __/__   _| |_) |
|_| |_|_|\__\___|_| |_|_| |_|_|_|\_\___|  |_| |_.__/


----------------------------------------------------------------------------------------------------

# Source Code

import os
os.environ["PAGER"] = "cat" # No hitchhike(SECCON 2021)

if __name__ == "__main__":
    flag1 = "********************FLAG_PART_1********************"
    help() # I need somebody ...

if __name__ != "__main__":
    flag2 = "********************FLAG_PART_2********************"
    help() # Not just anybody ...

----------------------------------------------------------------------------------------------------

Welcome to Python 3.10's help utility!

If this is your first time using Python, you should definitely check out
the tutorial on the internet at https://docs.python.org/3.10/tutorial/.

Enter the name of any module, keyword, or topic to get help on writing
Python programs and using Python modules.  To quit this help utility and
return to the interpreter, just type "quit".

To get a list of available modules, keywords, symbols, or topics, type
"modules", "keywords", "symbols", or "topics".  Each module also comes
with a one-line summary of what it does; to list the modules whose name
or summary contain a given string such as "spam", type "modules spam".

help> __main__
Help on module __main__:

NAME
    __main__

DATA
    __annotations__ = {}
    flag1 = 'ctf4b{53cc0n_15_1n_m'

FILE
    /home/ctf/hitchhike4b/app_35f13ca33b0cc8c9e7d723b78627d39aceeac1fc.py


help>
```

help utilityについて調べていると，[Python help() について - Opensourcetechブログ](https://www.opensourcetech.tokyo/entry/2018/05/08/Python_help%28%29_%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)に出会った。とりあえず，`modules`と実行してみると，

```
help> modules
...（略）...
_codecs_jp          aifc                imghdr              socket
_codecs_kr          antigravity         imp                 socketserver
_codecs_tw          app_35f13ca33b0cc8c9e7d723b78627d39aceeac1fc importlib           spwd
_collections        argparse            inspect             sqlite3
_collections_abc    array               io                  sre_compile
...（略）...
```

なんか明らかに怪しいやつがいた。
こいつを2回打ち込んでみると，

```
help> app_35f13ca33b0cc8c9e7d723b78627d39aceeac1fc
 _     _ _       _     _     _ _        _  _   _
| |__ (_) |_ ___| |__ | |__ (_) | _____| || | | |__
| '_ \| | __/ __| '_ \| '_ \| | |/ / _ \ || |_| '_ \
| | | | | || (__| | | | | | | |   <  __/__   _| |_) |
|_| |_|_|\__\___|_| |_|_| |_|_|_|\_\___|  |_| |_.__/


----------------------------------------------------------------------------------------------------

# Source Code

import os
os.environ["PAGER"] = "cat" # No hitchhike(SECCON 2021)

if __name__ == "__main__":
    flag1 = "********************FLAG_PART_1********************"
    help() # I need somebody ...

if __name__ != "__main__":
    flag2 = "********************FLAG_PART_2********************"
    help() # Not just anybody ...

----------------------------------------------------------------------------------------------------

Welcome to Python 3.10's help utility!

If this is your first time using Python, you should definitely check out
the tutorial on the internet at https://docs.python.org/3.10/tutorial/.

Enter the name of any module, keyword, or topic to get help on writing
Python programs and using Python modules.  To quit this help utility and
return to the interpreter, just type "quit".

To get a list of available modules, keywords, symbols, or topics, type
"modules", "keywords", "symbols", or "topics".  Each module also comes
with a one-line summary of what it does; to list the modules whose name
or summary contain a given string such as "spam", type "modules spam".

help> app_35f13ca33b0cc8c9e7d723b78627d39aceeac1fc
Help on module app_35f13ca33b0cc8c9e7d723b78627d39aceeac1fc:

NAME
    app_35f13ca33b0cc8c9e7d723b78627d39aceeac1fc

DATA
    flag2 = 'y_34r5_4nd_1n_my_3y35}'

FILE
    /home/ctf/hitchhike4b/app_35f13ca33b0cc8c9e7d723b78627d39aceeac1fc.py
```

出てきた。

## reversing
### Quiz[650 team solved, 50pt]
>クイズに答えよう!
>
>quiz.tar.gz a7f225b59176baa3d888c6fc7452f8df9a58e204

`file`コマンドでチェックしてみると，ELFというlinuxでの実行ファイルだったので，sshで，無料で立てていたubuntu on azureに繋いで実行してみた。すると，
```
$ ./quiz
Welcome, it's time for the binary quiz!
ようこそ、バイナリクイズの時間です!

Q1. What is the executable file's format used in Linux called?
    Linuxで使われる実行ファイルのフォーマットはなんと呼ばれますか？
    1) ELM  2) ELF  3) ELR
Answer : 2
Correct!

Q2. What is system call number 59 on 64-bit Linux?
    64bit Linuxにおけるシステムコール番号59はなんでしょうか？
    1) execve  2) folk  3) open
Answer : 1
Correct!

Q3. Which command is used to extract the readable strings contained in the file?
    ファイルに含まれる可読文字列を抽出するコマンドはどれでしょうか？
    1) file  2) strings  3) readelf
Answer : 2
Correct!

Q4. What is flag?
    フラグはなんでしょうか？
Answer : ?
flag length must be 46.
```
`strings`コマンドなるものを教えてくれた。
`strings quiz`を実行してみたら，
```
...（略）...
u+UH
[]A\A]A^A_
ctf4b{w0w_d1d_y0u_ca7ch_7h3_fl4g_1n_0n3_sh07?}
Welcome, it's time for the binary quiz!
Q1. What is the executable file's format used in Linux called?
...（略）...
```
フラグが転がってた。あとでquizに打ち込んでみても正解できた。

### WinTLS[102 team solved, 100pt]
>TLSってなんだぁ？
>
>wintls.tar.gz 4607f34efbcbc99137e684c00d7e4cb4425ec358

解けそうな問題がこれを始めた頃はなかったので，Macユーザーだけど，とりあえず調べてやってみた。基本は，[IDA Tutorial + CTF・脆弱性診断 超入門 Reversing③ 動画版 - 株式会社Ninjastars 技術系ブログ](https://www.ninjastars-net.com/entry/2019/03/11/090000)の動画の通りに進めた。さすがに全く同じではなく，パスワードをそのままチェックする部分が，
```
c4{fAPu8#FHh2+0cyo8$SWJH3a8X
tfb%s$T9NvFyroLh@89a9yoC3rPy&3b}
```
という2つの文字列に分かれていて，それぞれの部分で一致のチェックがされていた（問題修正前は片方だけだった？）。ただ，それぞれの長さが異なるように，単純に組み合わせれば良いわけではなさそうだった。
そのため，先ほどのサイトとほぼ同様にブレイクポイントを設定した上で，入力欄に`0123456789abcdefghijklmnopqrstuvxyz`と入力してみた。すると，

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.51.38.png)
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.51.51.png)

このように分かれた。終わったら，`aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa0123456789abcdefghijklmnopqrstuvxyz`でも試し，以下のようになった。

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.53.41.png)
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.53.59.png)

そしたらあとは，脳筋戦術で書き出して，

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.54.57.png)

-大文字小文字のタイプミスをするなどしながら- `ctf4b{f%sAP$uT98Nv#FFHyrh2o+Lh0@8c9yoa98$ySoCW3rJPH3y&a83Xb}`を得た。
正解！

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.56.43.png)

## crypto
### CoughingFox[443 team solved, 55pt]
>きつねさんが食べ物を探しているみたいです。
>
>coughingfox.tar.gz c2cdda5cb20d25e40be57a72a949591b7172d143

なんかの規則で暗号化してるみたいだった。

```python
from random import shuffle

flag = b"ctf4b{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX}"

cipher = []
print(flag)

for i in range(len(flag)):
    f = flag[i]
    c = (f + i)**2 + i
    cipher.append(c)

shuffle(cipher)
print("cipher =", cipher)
```
outputにあったリストは49文字。
```
cipher = [12147, 20481, 7073, 10408, 26615, 19066, 19363, 10852, 11705, 17445, 3028, 10640, 10623, 13243, 5789, 17436, 12348, 10818, 15891, 2818, 13690, 11671, 6410, 16649, 15905, 22240, 7096, 9801, 6090, 9624, 16660, 18531, 22533, 24381, 14909, 17705, 16389, 21346, 19626, 29977, 23452, 14895, 17452, 17733, 22235, 24687, 15649, 21941, 11472]
```

フラグが`ctf4b{[\x20-\x7e]+}`であることから，2乗しても少なくとも`(0x20^2=)32^2=1024`となり，隣の平方との差は少なくとも`(32+1)^2=1089`との差である65はあることがわかる。
リストの項目数は49個で，49文字だということが確認されているので，暗号化時の`i`は大きくとも48だとわかる。
よって，適当に0から平方の和をとって比較し，超える一つ前の値が`f+i`で，その時のiを差を取って求められるとわかった。
よって，あとは何の工夫もなく適当に汚いコーディングをして，
```python
cipher = [12147, 20481, 7073, 10408, 26615, 19066, 19363, 10852, 11705, 17445, 3028, 10640, 10623, 13243, 5789, 17436, 12348, 10818, 15891, 2818, 13690, 11671, 6410, 16649,
          15905, 22240, 7096, 9801, 6090, 9624, 16660, 18531, 22533, 24381, 14909, 17705, 16389, 21346, 19626, 29977, 23452, 14895, 17452, 17733, 22235, 24687, 15649, 21941, 11472]
answerstr = [‘a’] * len(cipher)
for c in cipher:
    ans = 0
    location = 0
    for i in range(200):
        if c < i ** 2:
            ansd = i - 1
            location = c - ansd ** 2
            ans = ansd - location
            answerstr[location] = chr(ans)
            print(f’ans: {ans}, location: {location}, chr(ans): {chr(ans)}’)
            break
answerstr = ‘’.join(answerstr)
print(len(cipher))
```

```
...（略）...
ans: 110, location: 43, chr(ans): n
ans: 111, location: 11, chr(ans): o
ans: 104, location: 28, chr(ans): h
ans: 89, location: 44, chr(ans): Y
ans: 115, location: 34, chr(ans): s
ans: 119, location: 38, chr(ans): w
ans: 101, location: 24, chr(ans): e
ans: 111, location: 37, chr(ans): o
ans: 84, location: 23, chr(ans): T
ctf4b{Hey,Fox?YouCanNotTearThatHouseDown,CanYou?}
```
`ctf4b{Hey,Fox?YouCanNotTearThatHouseDown,CanYou?}`と求まった。

## Welcome
### Welcome[845 team solved, 50pt]
Discordでフラグが配られた，参加賞。 ~~なぜここに書いた~~

## 終わりに
3人だったのですが意外となんか健闘して，74/891位となれました。初めてにしてはできすぎて怖いレベルです。pwnとかcryptとかgoとか色々，マジでわかんないやつ解いてくださって，マジすごかった。Playgroundの先輩方，ありがとうございました！ ~~1000点取りたかったねと残念がってる~~ 

<img width="987" alt="Screen Shot 2022-06-05 at 17 21 04" src="https://user-images.githubusercontent.com/41906969/172045010-0abad28c-42ca-4e6d-a7f9-f24e492145db.png">

