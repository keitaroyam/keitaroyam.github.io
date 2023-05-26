---
title: "Servalcatのインストール"
date: 2023-05-01T22:54:16+01:00
draft: false
tags: [servalcat]
---

[Servalcat](https://github.com/keitaroyam/servalcat) 0.4.6からC++コードを含むようになり，少々配布方法が複雑になってしまった．
C++側からもPython側からもGEMMIが必要なため，(Pythonライブラリとしての) GEMMIと同じコンパイラでビルドしなければGEMMIオブジェクトの互換性が失われる．
[PyPI](https://pypi.org/project/servalcat/)上で配布している(pipでインストールされる)バイナリは本家GEMMIと同一方法でビルドしているので，パッケージ版は問題なく動くはず．
ただし特殊なプラットフォームでは自力でのビルドが必要になる．またPython 3.6以前を使う場合も自力でのビルドが必要である．

## パッケージ版
上述のように，
```sh
python -mpip install -U seravalcat
```
で入るはず．ただしここでビルドが始まった場合(= 環境に合う配布バイナリが無かった)は，一見うまくいってもGEMMIを配布バイナリからインストールしてる場合は動かない．
下記の「自分でビルドする場合」を参照．

## Github最新版
Github最新版 (パッケージ版に未収録)をインストールする場合，Github Actionsで自動ビルドされたバイナリを使うことができる．
1. https://github.com/keitaroyam/servalcat/actions/workflows/ci.yml のworkflow runsのうち，最新のrunをクリック
2. Artifactsから"wheels2"をダウンロード
3. zipを解凍し，以下のコマンドを実行． `wheels2`はzipの中身が展開された場所(ディレクトリ)の名前
```sh
python -mpip install -U --find-links=wheels2/ servalcat
```

## 自分でビルドする場合
まずパッケージ版のGEMMIを自分でビルドする．
Github最新版のGEMMIは動かない可能性があるので注意(Servalcatはパッケージ版の最新のGEMMIで動くように作っている)．
どのようにしても良いが，pipに `--no-binary` オプションがあるのでそれが使えるかも（未検証）
```sh
python -mpip install --force-reinstall --no-binary gemmi gemmi 
```
それからGithub最新版のServalcatをソースからビルド
```sh
python -mpip install -U git+https://github.com/keitaroyam/servalcat.git
```
これで動くはず(未検証)
