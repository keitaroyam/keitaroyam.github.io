---
title: "Cootに元素を変更する機能を追加する"
date: 2023-05-30T19:06:08+01:00
tags: [coot]
---

とりさんから一年前に約束した当該機能まだですかと聞かれ，全く身に覚えが無かったものの，実装したらDVD買ってあげるということで書きました．

Coot 0.9.8.7で動作確認．

```py
def change_elem_ext():
    def change_elem(atom_spec, elem):
        imol, chain, resi, ins, name, alt = atom_spec[1:]
        set_atom_string_attribute(imol, chain, resi, ins, name, alt, "element", elem)

    def f(elem):
        add_status_bar_text("Click an atom")
        user_defined_click(1, lambda x: change_elem(x, elem))

    generic_single_entry("Element?", "", "OK", lambda text: f(text))

coot_toolbar_button("Change elem", change_elem_ext, None)
```

上記pythonコードを適当な名前(.py)で保存して Calculate - Run Script から実行するか，~/.coot-preferences 以下に置いておくと起動時に読まれます．
成功すればtoolbarに"Change elem"のボタンが追加されます．

ちなみにこういったスクリプトを書くときにAPIを調べる方法ですが，体系的には把握してないので毎回ソースコードから探してます．
Python側で定義されてる場合は.pyのファイルにあるのでそこから探し，今回は見つからなかったので src/coot_wrap_python.cc の方を見てatomでgrepしたらset_atom_attributeを見つけました．
これは実数値を変更する関数だったのですが，下の方にset_atom_string_attributeを発見し，attribute_nameとしてelementを受け付けてくれることが分かりました．
https://github.com/pemsley/coot/blob/abe4c5d9e372b91ba43d4bd9a7e924042c7f00ff/src/molecule-class-info-other.cc#L550

文字列をユーザに入力させるとか，クリックした原子の情報を取得するとかいった部分は，類似の他の機能のコードを探して真似します．
特に拡張的な機能はたいていpythonで書かれています．