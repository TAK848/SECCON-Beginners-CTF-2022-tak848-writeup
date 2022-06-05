# SECCON Beginners CTF 2022
## はじめに
SECCON Beginners CTF 2022 [SECCON Beginners CTF 2022 を開催いたします！ - SECCON2022](https://www.seccon.jp/2022/seccon_beginners/content.html)に，playgroundというコミュニティの先輩方と3人で参加したので，僕が関わった問題についてWriteUPします。
CTFというものが完全に初めてだったので試行錯誤でしたが，意外と個人的には奮闘できたと思います。
## Web
### textex[128 team solved, 92pt]
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.11.39.png)
愉快なツールが登場した。
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.12.37.png)
ファイルはこんな感じで，`flag`を読み込めればよさそうだ。

TeXでは`\input`によりファイルの中身をそのまま読み込める。
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

最後に，フラグに何らかのLaTeXの特殊文字が含まれていることがわかったので，そんな時はと`listings`を試したがインポートできなかった。調べてみて，`verbatim`をインストールすることに成功したので，最終的に
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
入力したテキストを，`Murecho`というオープンなフォントで画像ファイルに書き出し，english指定でocrにかけ直していた。
このocrに掛け直した文字が，`www.example.com`のどの文字も使わずに`www.example.com`とOCRを騙すことができればflagが帰ってくるようになっていた。
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.26.55.png)
-実験レポで全角半角変換でお世話になっている- [Unicodeの一覧ツール](https://so-zou.jp/web-app/text/unicode/)をテキストエディタに貼り付け，Murechoフォントで表示した。その後はひたすら気合いで探しては試行錯誤を繰り返して，何とか`ωωω․εⅹáⅿρIε․ċοⅿ`という文字列でうまくひっかっかった。
ドットを探すのが特に大変だった。
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.30.45.png)
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.30.30.png)
[文章比較ツール](https://lab.hidetake.org/diff/)より。みなさんも，フィッシングに釣られないよう気をつけましょう。
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.00.23.png)

### hitchhike4b[125 team solved, 84pt]
コードなしでの取得だった。先輩が`__main__`と入力するとフラグの前半がもらえることがわかり，後半を探した。
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.37.23.png)
help utilityについて調べていると，[Python help() について - Opensourcetechブログ](https://www.opensourcetech.tokyo/entry/2018/05/08/Python_help%28%29_%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)に出会った。とりあえず，`modules`と実行してみると，
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.39.20.png)

なんか明らかに怪しいやつがいた。
こいつを2回打ち込んでみると，
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.39.55.png)
出てきた。（時間内になんで2回打ったのかはよく覚えていないが，リセットされたと思って打ったのかな？）
## reversing
### Quiz[650 team solved, 50pt]
`file`コマンドでチェックしてみると，ELFというlinuxでの実行ファイルだったので，sshで，無料で立てていたubuntu on azureに繋いで実行してみた。すると，
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.42.55.png)
`strings`コマンドなるものを教えてくれた。
`strings quiz`を実行してみたら，
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2016.43.39.png)
フラグが転がってた。あとでquizに打ち込んでみても正解できた。

### WinTLS[102 team solved, 100pt]
解けそうな問題がなかったので，Macユーザーだけど，とりあえず調べてやってみた。基本は，[IDA Tutorial + CTF・脆弱性診断 超入門 Reversing③ 動画版 - 株式会社Ninjastars 技術系ブログ](https://www.ninjastars-net.com/entry/2019/03/11/090000)このサイトの動画の通りに進めた。さすがに全く同じではなく，
```
c4{fAPu8#FHh2+0cyo8$SWJH3a8X
tfb%s$T9NvFyroLh@89a9yoC3rPy&3b}
```
という2つの文字列に分かれていて，一致のチェックがそれぞれされていた（問題修正前は片方だけだった？）。ただ，それぞれの長さが異なるように，単純に組み合わせれば良いわけではなさそうだった。
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
なんかの規則で暗号化してるみたいだった。
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.02.17.png)
49文字。
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.07.16.png)

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

![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.18.38.png)
`ctf4b{Hey,Fox?YouCanNotTearThatHouseDown,CanYou?}`と求まった。
## Welcome
### Welcome[845 team solved, 50pt]
Discordでフラグが配られた，参加賞。 ~~なぜここに書いた~~

## 終わりに
3人だったのですが意外となんか健闘して，74/891位となれました。初めてにしてはできすぎて怖いレベルです。pwnとかcryptとかgoとか色々，マジでわかんないやつ解いてくださって，マジすごかった。Playgroundの先輩方，ありがとうございました！ ~~1000点取りたかったねと残念がってる~~ 
![](SECCON%20Beginners%20CTF%202022/Screen%20Shot%202022-06-05%20at%2017.21.04.png)

#ctf
