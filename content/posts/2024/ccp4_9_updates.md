---
title: "CCP4 9.0アップデートまとめ"
date: 2024-06-12
tags: [ccp4]
ShowToc: true
---

私の興味範囲のみのまとめです．
公式情報: https://www.ccp4.ac.uk/ccp4-8-0-updates/

リリースノートに記載されてないアップデートも実際にはあるのがCCP4 update.
実際にどのファイルが変わったのかは $CCP4/restore/update.log に記録されています．

## 9.0.005
2024-12-10公開

### Acedrg 306 (rev 347)

* link
* 金属含有化合物への対応(途中)

## 9.0.004
2024-10-19公開

### Acedrg 298 (rev 330)

* link作成時にplane/torsionの名前が重複する問題の修正
* デフォルトのmonomer IDがUNLからLIGに変更
* 金属含有化合物への対応(途中)

### Servalcat 0.4.88

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.88

refmacatで `--keep_entities` 指定されたときにinputのentityをoutputにもそのままコピーするように変更 (PDB-REDOからのリクエスト)．

## 9.0.003
2024-08-19公開

### Servalcat 0.4.82

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.82

refmacatに関しては，unrestrained refinementのときに未知のlink idがあると止まってしまうバグを修正．
もう一つ，352dのP/156のようなaltlocで原子の組成が異なるモデルに対してうまくいかないバグがあり，これはgemmi側で修正済みなので次のバージョンから対応される．
これでPDBの構造に対してunrestrained refinementは全部うまくいくはず．

あとはtwin refinementの実装を追加 (experimental)

### Mosflm

mosaicity計算のバグが修正された．レポジトリの方でコミットが見つからない．．

## 9.0.002
2024-07-24公開

### Refmac 5.8.0430 (rev 497)

バージョン番号変わってませんが，mmcif/crdから\_struct_ncs_operから回転行列を読むときに，-1未満の数字があると行列の全要素が0になるバグが修正されています．例えば3cjiの31番目の要素が
```
31 generate 0.999999 -0.000494 0.000078 0 -0.000483 -0.999992 0.000166 179.67 0.000075 -0.000165 -1.000002 253.383
```
となっており，これが全部0になってしまうという問題．pdbからMTRIXを読む場合はもともと問題ありませんでした．
Refmacatの場合はcrd (cif)を経由するので，この問題の影響を受けてしまってました．

### Servalcat 0.4.77

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.77

refmacatに関する修正が主．
* "given”のstrict NCS matrixを使わないように修正 (ただし出力ファイルから漏れてしまう問題が残ってる)
* 出力mmcifのlabel_seq_idを修正
* 未知のexternal restraint keywordがあると落ちる問題の修正(exte torsでperiodを指定した人がいて発覚)
* unrestrained refinementのキーワードで大文字小文字の区別が入ってしまっていたので修正 + unrestrainedの場合でもcrdを経由するように修正
* 出力mmcifのrefmac versionに(refmacat XXX)の記述を追記

PDB側でもマップ生成にrefmacの代わりにrefmacatを使い始めたようで，unrestrained refinementに関する修正はその目的．
unrestrained + 0 cycleなのでrefmacatの意味無いと思っていたが，refmac5に直接4icmのcifを読ませると落ちるなど問題がある模様．


## 9.0.001
2024-07-16公開

### Coot 0.9.8.95

https://www.mail-archive.com/coot@jiscmail.ac.uk/msg05701.html

これが最後の0.9系になるかもしれない？


### Acedrg 293

バージョン番号は変わっていませんが，linkのcif fileが壊れる問題が修正されました．

### Servalcat 0.4.74

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.74

refmacat的には，
* exte指定でsymm指定があると無限ループに陥る問題の修正
* 勝手にmetal linkを追加しないように修正 (make link y時のみ実行)


## 9.0.000
2024-06-11公開

Python 3.9．

実質release candidate扱いで，次のminor updateで広くアナウンスするとのこと

### DIALS 3.19

CCP4 8.0がPython3.7だったのでずっとDIALS 3.8 (2022-01-11リリース)で止まっていたのが，ついに最新版に更新．

### Acedrg 293

Acedrg 277から色々と変わりました．主な更新は以下の通り．

* Refmacではなくservalcat refine_geomでideal coordinatesを生成するように変更
* 金属含有化合物に対応するための変更 (metalCoordの使用が前提)
* \_chem\_comp_atom.alt\_atom\_id の追加 (古い原子名．gemmiはこれを利用してモデルを修正できる)
* \_chem_comp_ring_atom の追加
* cif tagの変更
  * \_chem\_comp\_bond.type → \_chem_comp_bond.value_order
  * \_chem_comp_bond.aromatic → \_chem_comp_bond.pdbx_aromatic_flag
  * \_pdbx_chem_comp_description_generator → \_acedrg_chem_comp_descriptor
* M07でN1-O7, C8-N1周りのtorsion angleがconstになる問題の修正 (sp2_sp2)
* MSEのchi2の名前が正しくつくように修正
* PROのようなアミノ酸の主鎖Nをデフォルトでプロトン化するように変更
* eLBOWの出力などで原子名が’を含む場合にAcedrgがコケる問題の修正
* アミンのchiral指定が入らないように修正 (T0NのNC1, NC2など) 参考: [Chirality at Nitrogen, Phosphorus, and Sulfur](https://chem.libretexts.org/Bookshelves/Organic_Chemistry/Organic_Chemistry_(Morsch_et_al.)/05%3A_Stereochemistry_at_Tetrahedral_Centers/5.10%3A_Chirality_at_Nitrogen_Phosphorus_and_Sulfur)
* CO2の結合角が165°になってしまう問題の修正

### LIBG

external restraintに無用に"type"が設定されていたバグが修正されました．
以前はこれのせいで水素結合の距離制約がbond length扱いになってしまい，bond length rmsdがインフレしてしまう問題がありました

### Coot 0.9.8.94

https://www.mail-archive.com/coot@jiscmail.ac.uk/msg05671.html

糖鎖linkに関する修正．

### monomer library

注意: Mac版ではなぜか古いmonomer libraryのままになっています．9.0.001で修正．

https://github.com/MonomerLibrary/monomers/commits/ccp4-9.0.000

ccp4-8.0.018以降の更新 (019では古いのに戻されてしまったので)は以下の通り

1. 長いmonomer IDの例としてA1LU6を追加 [#43](https://github.com/MonomerLibrary/monomers/pull/43)
2. mon\_lib\_list.cifからlinkパートを取り出したlinks\_and\_mods.cifの追加 [#49](https://github.com/MonomerLibrary/monomers/pull/49)
3. 水素化物(XH<sub>n</sub>)の修正 [#52](https://github.com/MonomerLibrary/monomers/pull/52)
4. O-glycosylationのad hocな修正 [#55](https://github.com/MonomerLibrary/monomers/pull/55)

3.では，水 (HOH)やアンモニア (NH3)が重要な修正です．
あろうことか，いずれも結合角が120°になっていて，NH3は平面分子になってしまっていました．
今まで水素化物はRefmacによる最適化がコケるせいで，Acedrgによる更新から漏れてしまっていました．
今回Acedrgがservalcatをgeometry optimisationに使うようになったので更新できるようになりました．
中性子の精密化では重要な修正になります．

4.は，O-glycosylation (pyr-SERとpyr-THR)のlinkにおいて，角度のrestraintが不足していたためジオメトリが容易に歪む問題を修正しています．
また，O-glycosylationにおいてはchiralが固定ではないので，chiralをbothに変更しています．
chiralを適切に設定するには糖のグループ分けから再考する必要があり，この変更はとりあえずの応急処置です．

### Refmac 5.8.0430

* ログ出力においてRfactorsが化けてしまう(\*\*\*)問題の修正
* NCS restraintの情報が正しくmmcifに書かれていなかった問題の修正

### Servalcat 0.4.72

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.72

refmacat的には，ほぼRefmacとの互換性問題の修正です．

* Refmacと同様に1を超えた占有率を1に置き換えるように変更
* monomer libraryの情報を使って元素名を自動修正
* mmcif出力でblock名が重複しうる問題を回避
* MTRIX情報をcrdに書き込み，”ncsc”指定時にヘッダ情報の行列を使って機能するように修正
* chain IDが無い場合でも動くように修正
