Doxygen の LaTeX 出力で日本語を使う（LaTeX スタイルシートで）
======================================================

- 参考 URL:
    - [イグトランスの頭の中（のかけら）: 2011年版](https://dev.activebasic.com/egtra/2011/12/25/459/)
    - [イグトランスの頭の中（のかけら）: 2015年版](https://dev.activebasic.com/egtra/2015/06/29/814/)
    - [LaTeX の「アレなデフォルト」傾向と対策](https://qiita.com/zr_tex8r/items/297154ca924749e62471)
    - [TeX Wiki / hyperref](https://texwiki.texjp.org/?hyperref)

概要
----

参考 URL の最初の2つのサイトにあるように、Doxygen の LaTeX 出力ソースからの PDF 作成は、
欧文の LaTeX もしくは pdfLaTeX を想定しており、そのままでは日本語が使えません。
PDF の日本語が文字化けしてしまいます。

そこで上記参考 URL のサイトを参考にして、同様のことを行う LaTeX スタイルシートを作ってみました。
参考サイトでは、Doxygen が出力した LaTeX ソースファイル（refman.tex）を sed で編集しています。
Doxygen を実行するたびに修正するのは、ほんの僅かではありますが手間がかかります。

以降で説明するスタイルシートでは、ファイル名を Doxygen の Doxyfile に書いておくことで、
出力されたファイルを修正することなしに、日本語の PDF を作成することができるようになります。

また、私の環境だけかもしれませんが、 Doxygen の出力から作成した PDF では、
関数名などのリンクをクリックしたときに、その定義場所にジャンプできていませんでした。
その現象の修正もスタイルシートで行っています。

- - - - - - - - -

動作環境
-------

動作確認は以下の環境で行いましたが、他の環境でも同じスタイルシートが使えると期待しています。

- オペレーティングシステム： Windows 7 Professional 64bit
- アプリケーション：
    - Doxygen: 1.8.14 (Cygwin 版)
    - LaTeX: e-pTeX 3.14159265-p3.8.1-180226-2.6 (utf8.euc) (TeX Live 2018/Cygwin)

- - - - - - - - -

Doxygen の設定
--------------

プロジェクト用の Doxyfile に、以下の設定を行います。

### GENERATE_LATEX

    GENERATE_LATEX = YES

LaTeX 出力を有効にします。

### LATEX_CMD_NAME

    LATEX_CMD_NAME = uplatex

LaTeX のコマンド名に、日本語に対応した upLaTeX のコマンドを指定します。

### USE_PDFLATEX

    USE_PDFLATEX = NO

pdfLaTeX は日本語を扱えないため、これを使わないようにします。

### LATEX_EXTRA_STYLESHEET

    LATEX_EXTRA_STYLESHEET = doxygenjpext.sty

以降で説明するスタイルシートファイルを指定します。
Doxygen が出力する LaTeX ソースファイルの冒頭に、
`\usepackage{doxygenjpext}` が挿入されるようになります。

### PDF_HYPERLINKS

    PDF_HYPERLINKS = YES

PDF で HTML 出力のようなハイパーリンクを、ページ参照の代わりに出力するようにします。
Adobe Acrobat Reader のような PDF Viewer でブラウズするときに便利です。

- - - - - - - - -

スタイルシートの用意
-------------------

以降で説明するスタイルシートファイルを、Doxyfile と同じディレクトリに置きます。

スタイルシートファイルは、 GitHub からダウンロードしてください。
ファイル名は [`doxygenjpext.sty`](doxygenjpext.sty) です。

- - - - - - - - -

PDF の作成
----------

準備ができたら、以下の手順で PDF を作成します。

    cd /path/to/your/project
    doxygen
    cd doc/latex
    make && dvipdfmx refman

- - - - - - - - -

内容の説明（何をしているのか）
---------------------------

### パッケージに渡すドライバを指定する

各種パッケージに、 `dvipdfmx` または `dvipdfm` ドライバを指定します。

    \PassOptionsToClass{uplatex,dvipdfmx}{book}
    \PassOptionsToPackage{dvipdfmx}{adjustbox}
    \PassOptionsToPackage{dvipdfmx}{color}
    \PassOptionsToPackage{dvipdfmx}{xcolor}
    \PassOptionsToPackage{dvipdfmx}{graphicx}
    \PassOptionsToPackage{dvipdfm}{geometry}

### フォントメトリックの調整

和文フォントのメトリックを調整するために、`otf` パッケージなどを読み込みます。

    \usepackage[deluxe,jis2004,uplatex]{otf}
    \usepackage[T1]{fontenc}
    \usepackage{lmodern}

### 本文をゴシック体に変更

Doxygen は書体にサンセリフ体を指定していますが、
和文の書体は指定していないためデフォルトの明朝体が使われてしまい、
欧文と和文がアンバランスになってしまいます。
そこで、本文の和文書体をゴシック体に変更します。

    \renewcommand{\kanjifamilydefault}{\gtdefault}

### 見出しなどの和文書体をデフォルトの太字にする

Doxygen の出力に合わせて、見出しが太字になるようにします。

    \DeclareFontShape{JY2}{hgt}{bc}{n}{<->ssub*hgt/bx/n}{}
    \DeclareFontShape{JT2}{hgt}{bc}{n}{<->ssub*hgt/bx/n}{}

### PDF のハイパーリンク対応

#### 和文対応としおり（ブックマーク）対応

Doxygen の出力した LaTeX ファイルでは、`hyperref` パッケージに `pdftex`
または `ps2pdf` ドライバが指定されています。これを `dvipdfmx` に変更します。

まず、`hyperref` パッケージを `dvipdfmx` ドライバを指定して読み込んで、
さらに和文に対応させるために、`pxjahyper` パッケージを読み込みます。

そして、このスタイルシート以降に現れた `\usepackage` に `hyperref` が指定されていたら、
無視するようにしてしまいます。

    \RequirePackage[dvipdfmx,pagebackref=true]{hyperref}
    \usepackage{pxjahyper}
    %
    \RequirePackage{xifthen}
    \RequirePackage{letltxmacro}
    \LetLtxMacro\originalusepackage\usepackage\relax
    \renewcommand\usepackage[2][]{%
      \ifthenelse{\equal{#2}{hyperref}}{}{\originalusepackage[#1]{#2}}%
    }


また、`hyperref` パッケージに指定されている `unicode` オプションを `false` に指定しなおします。
これは、 `unicode` オプションが (u)pLaTeX に対応していないためです。

Doxygen の出力では、このスタイルシートを読み込んだ後に `hyperref` パッケージを読み込んでおり、
その直後に `hypersetup` コマンドでオプションを指定しているため、
プリアンブルの読み込みが終わってから指定し直すようにします。

    \AtBeginDocument{%
      \hypersetup{%
        unicode=false%
        bookmarks=true,bookmarksnumbered=true,% enable bookmarks
        setpagesize=false}%
    }

#### Doxygen の出力のハイパーリンクを修正する

私の環境のせいかもしれませんが、 Doxygen の出力から作成した PDF では、
関数名などのリンクをクリックしたときに、その定義場所にジャンプできていませんでした。
これはどうやら、Doxygen がリンクの参照に `\hyperlink` コマンドを使用しているせいのようです。
`\hyperlink` コマンドは、`\hypertarget` コマンドを使って参照先を設定するもののようですが、
Doxygen は参照先に `\label` コマンドを使用しているので、
dvipdfmx が PDF に変換するときに参照先が見つからないというエラーになってしまいます。

そこで `\hyperlink` コマンドを `\hyperref` コマンドにマクロ置換するようにします。
しかしながら、目次や索引の参照は `\hyperref` に置き換えるとリンクができないようでした。
そのため、 プリアンブルの `\makeindex` には影響がないようにするとともに、
`\tableofcontents` と `\printindex` の前後では、
マクロ置換を元に戻すようにしました。

それらを行うマクロが以下のようになります。

    \AtBeginDocument{%
      \LetLtxMacro\originalhyperlink\hyperlink\relax
      \def\hyperlinkexch#1#2{\hyperref[#1]{#2}}
      \LetLtxMacro\originaltableofcontents\tableofcontents\relax
      \def\tableofcontents{%
        \let\hyperlink\originalhyperlink\originaltableofcontents%
        \let\hyperlink\hyperlinkexch}%
      \LetLtxMacro\originalprintindex\printindex\relax
      \renewcommand\printindex{%
        \let\hyperlink\originalhyperlink\relax\originalprintindex[default]%
        \let\hyperlink\hiperlinkexch}%
    }

- - - - - - - - -

最後に
-----

以上のマクロの断片をつなぎ合わせて、 `doxygenjpext.sty` に保存すれば完成です。

今回、このスタイルシートを作成する際には、冒頭の参考 URL のサイトの他にも
様々なサイトを参考にさせていただきました。

先人の方々には、大変感謝いたします。
