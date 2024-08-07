---
title: "CCP4 8.0アップデートまとめ"
date: 2023-06-14
tags: [ccp4]
ShowToc: true
---

私の興味範囲のみのまとめです．
公式情報: https://www.ccp4.ac.uk/ccp4-8-0-updates/

リリースノートに記載されてないアップデートも実際にはあるのがCCP4 update.
実際にどのファイルが変わったのかは $CCP4/restore/update.log に記録されています．

## 8.0.019
2024-04-11公開

### monomer library

下記の問題のために古いバージョンに戻されてしまった

https://github.com/keitaroyam/servalcat/issues/14


## 8.0.018
2024-04-03公開

新しいServalcatが入るはずでしたが，GEMMIのビルドシステム変更のためにGEMMIのバージョンを上げられず，
最新のGEMMIを必要とするServalcatも次のCCP4メジャーアップデートまで見送りとなりました．
このためAcedrgのアップデートも見送りとなりました．
既知のバグが直ってないままになるのは残念だけど仕方ない．早く次のバージョンが出ますように

### Coot 0.9.8.93

https://www.mail-archive.com/coot@jiscmail.ac.uk/msg05645.html

* Mutate Residue RangeでDNAがRNAになってしまう問題の修正
* TYRのHH原子に関する問題の修正: https://github.com/pemsley/coot/commit/5b52df6509afd137d72c604a6dde1232fe6cff70

### monomer library

https://github.com/MonomerLibrary/monomers/commits/ccp4-8.0.018

1. AX/RXを追加．phaserで使われる未知の異常散乱/実原子．なぜか今までのRefmacでは定義無しで動いていたらしい？ [#51](https://github.com/MonomerLibrary/monomers/pull/51)
2. CCDにおける原子名の変更などを反映 [#44](https://github.com/MonomerLibrary/monomers/pull/44)
3. TYRのhh1 (OHのねじれ角)が間違っていたのを修正 [#45](https://github.com/MonomerLibrary/monomers/pull/45)
4. PROのNがsp2になってしまっていた問題の修正．すべてのP-peptideをプロトン化(NH2+)状態に [#38](https://github.com/MonomerLibrary/monomers/pull/38)
5. B12の修正 [#41](https://github.com/MonomerLibrary/monomers/pull/41)
6. PDBによるペプチド主鎖原子の修正を反映 [#40](https://github.com/MonomerLibrary/monomers/pull/40)
7. Robbieによる修正 [#35](https://github.com/MonomerLibrary/monomers/pull/35)


## 8.0.017
2024-01-18公開

### Refmac 5.8.0425

http://fg.oisin.rc-harwell.ac.uk/scm/loggerhead/refmac/5.8/revision/487

1. 残基数の最大値を50000から100000に
2. symmetry related external bond distanceが正しく機能しないバグを修正

LINKヘッダに書く順序で認識されたりされなかったりするバグが発生しており，直っていない．Refmacatを使えば問題なし．

### CCP4i2

Refmac5インタフェースでRefmacatが使われるようになり，[中性子構造の精密化](https://doi.org/10.1107/S2059798323008793)のインタフェースが追加された．

残念ながらServalcatが古いままで，最近行った様々なバグ修正が反映されてない．
[Windowsでgemmi 0.6.4が正しく動かないバグ](https://github.com/dials/dials/pull/2578)が見つかったことが原因．

### doubleHelix

新プログラム
https://doi.org/10.1093/nar/gkad553


## 8.0.016
2023-10-05公開

### Coot 0.9.8.92

https://www.mail-archive.com/coot@jiscmail.ac.uk/msg05575.html

前回のad hocな解決方法ではDNA-SERやdisulfでも問題が起きることが分かったので，ついに[ちゃんとした修正](https://github.com/pemsley/coot/commit/7e2518b1452f18d011db43b57210bce71cfa41e5)が入りました．
一部まだ挙動が怪しいようですが．

### monomer library

https://github.com/MonomerLibrary/monomers/commits/ccp4-8.0.016

1. ホスホジエステル結合(p)にtorsion angle alphaが抜けていたので追加 [#33](https://github.com/MonomerLibrary/monomers/pull/33)
2. Acedrg 278で20種のアミノ酸を更新 [#32](https://github.com/MonomerLibrary/monomers/pull/32)
3. すでにCCD (PDBの低分子ライブラリ)に存在しないエントリを削除 [#30](https://github.com/MonomerLibrary/monomers/pull/30)
4. gemmi 0.6.0以前でエラーになる項目を修正 [#29](https://github.com/MonomerLibrary/monomers/pull/29)

2.は一部のアミノ酸でねじれ角に不適切な理想値が設定されてしまっていた問題の修正です．特にMETやLEUなど．chi\*の名前はREFMACでも使われるので，ちょっと大きな問題でした．同時に，nucleus distanceも最新のテーブルの値になっています．

4.の件，そんな古いgemmi誰も使ってないだろうと思ってたんですが，[ccpem 1.6 (現状の安定版)でいまだにgemmi 0.5.3が使われている](https://www.jiscmail.ac.uk/cgi-bin/wa-jisc.exe?A2=ind2308&L=CCPEM&O=D&P=35596)ことが判明しました．
つまりccpem + ccp4最新版ではしばらく構造精密化が動かないままだったということに．．
何だかあんまりな話ですが，とりあえず今回の修正で解決されます．


## 8.0.015
2023-08-16公開

cloud関連の更新のみ

## 8.0.014
2023-08-15公開

### acedrg 277

Link作成時に結合をDELETEした際，その結合周りのbond angle restraintsなどが適切に削除されていなかった問題の修正

### Coot 0.9.8.91

https://www.mail-archive.com/coot@jiscmail.ac.uk/msg05544.html  
https://www.mail-archive.com/coot@jiscmail.ac.uk/msg05536.html

以下に書いた通り，RNA-タンパク複合体問題がついに解決．ただしad-hocな方法のままです．これDNA-SERとかで問題になるんじゃないだろうか…

あとはReplace residue的な機能でchain IDが変わってしまうバグも直ったらしい（たぶん10年以上存在していた）．

### servalcat 0.4.32

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.32

CCP4的に重要な修正は，LINKヘッダに存在しない原子が記述されていた場合にクラッシュしてしまう問題の修正．これでRefmacatの機能が問題なく使えるはず．ModelCraftはすでにrefmacatを使用．

一部で宣伝していた，refine\_xtal\_norefmacで低分子を精密化するとき大きめにrandomiseしてもちゃんと収束する性能が残念ながら失われてしまった．
これはADP restraintの方式を変えたせい．いちおう `--adp_restraint_mode diff --adpr_weight 3.5` で以前の挙動に戻せる．
自分で実装してはじめて，ADP restraintsが実は収束性能にかなり重要ということを知った．


## 8.0.013
2023-07-20公開

pandasのバージョンが0.24.1から1.1.5に．

### Refmac5.8.0419

http://fg.oisin.rc-harwell.ac.uk/scm/loggerhead/refmac/5.8/revision/482

1. 一部の水素だけを精密化したときにcalc_flagが全部"."になってしまっていた問題の修正
2. 水素原子のADP restraintで親原子のBに対する因子を調節可能に
3. Windowsでrefmacat動かない(make cr prepared時に動かない)問題の修正

2について補足．ADP精密化では近隣の原子のBとの差が小さくなるように制約されますが，水素の場合についてのみ，制約関数内部において親原子のB値をスケールできるようになりました
```
hydrogen bvalue factor <value>
hydrogen bvalue factor polar <value>
hydrogen bvalue factor nonpolar <cvalue> 
```

SHELXLも似たようなことをしていて，例えば親原子のU値の1.2倍に拘束するといったことをしますが，Refmacの場合は拘束ではなく制約という違いです．

### acedrg 276

芳香族性の情報のみから（水素原子なしで）構造を生成するときのバグ修正とかそんな感じ（あやふや）


## 8.0.012
2023-06-13公開

### Refmac5.8.0415

http://fg.oisin.rc-harwell.ac.uk/scm/loggerhead/refmac/5.8/revision/478

1. van der Waals, ion半径の調節が可能に
2. refmacatの場合にexternal restraintsのchain IDが正しくinstructionから読めていなかった問題の修正 (exte nonb, exte harm)
3. exte harm chai.. の指定が機能していなかった(exte harm atin chai...とする必要があった)ので修正

1について補足．これはカリウムの反発問題に対するとりあえずの修正です．
カリウムのvdwrは大きく，ener_lib.cifには2.75 Åとして定義されています．
一方，たとえばPDB 4ua6のA/2003を見てみると，K-Hの距離が3.33 (A/132 HD21), 2.97 (A/130 HB3) Åになっていて，Refmacはこれらを異常接近として見てしまいます (Hのvdwrは1.2 Åなので臨界距離=vdwr半径の和が3.95 Å，後者はOGとKにリンクがあるので片方は1-4の関係となるため0.3引いて3.65 Å)．
静電相互作用ネットワークを解析するなどすれば適切な臨界距離が出せるかも知れませんが，それはそれで大変なので，とりあえずKのvdw半径を小さくしてしまうことで対応できるようにしました．
`nonb maxradius vdw 2.0` とすれば，すべての原子についてvdw半径の最大値を2.0 Åにできます．
`exte nonb elem k vdwr 2.0` とすれば，K原子のvdw半径を2.0 Åに設定できます．


### Coot 0.9.8.8

https://www.mail-archive.com/coot@jiscmail.ac.uk/msg05507.html

核酸-タンパク複合体精密化の問題が直ってない…！
ただ，原因は分かりました．AA-RNA linkが悪さをしていました．
Cootのmake_other_types_of_link()のロジックはどうやら，2つの残基間の最小距離が3 Å以下かつ，その2つの残基間で可能なリンクが存在する場合，そのリンクをかけてしまうようです（実際に結合する原子間の距離を見ていない！）
だいぶまずいロジックだと思うんですが，応急処置としてAA-RNA (とSS)についてのみ結合する原子間の距離チェックを入れたようです(0.9.8.9から反映): https://github.com/pemsley/coot/commit/9bb39c34a884326f9cc734e2efb2d25ca03e08a6
ただこの方法だと，将来他の似たリンクが定義されたときにまた同じ事故が起こるので，根本的に解決して欲しいところです．．
開発者による解説記事: https://pemsley.github.io/coot/blog/2023/06/05/AA-RNA.html

### monomer library

https://github.com/MonomerLibrary/monomers/commits/ccp4-8.0.012

- DNA/RNAのモノマーはideal torsion angleとして2'-endo/3'-endoの両方の値を持ってるが，当該原子が存在しない場合でも機械的に定義が書かれてしまっていたのを修正 [#27](https://github.com/MonomerLibrary/monomers/pull/27)
- metals.jsonの追加 [#28](https://github.com/MonomerLibrary/monomers/pull/28)
- 矛盾したリンク定義類を修正 [#26](https://github.com/MonomerLibrary/monomers/pull/26)

metals.jsonはCODから作られた，金属と配位子間の理想距離情報（配位数依存）．今のところアルカリ金属およびアルカリ土類金属のみカバー．とりあえずこれでRefmacのMg-O距離が不適切な問題に対応できる．Servalcat/Refmacatからのみ利用可能．

### gemmi 0.6.2

https://github.com/project-gemmi/gemmi/releases/tag/v0.6.1  
https://github.com/project-gemmi/gemmi/releases/tag/v0.6.2

### servalcat 0.4.24

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.15  
https://github.com/keitaroyam/servalcat/releases/tag/v0.4.24

こっそり更新．上述のmetals.jsonの利用など．PB氏にテストしてもらったところ，LINKヘッダに存在しない原子が定義されている(gap含む)と例外吐いて落ちる問題が発覚…．0.4.25で修正．

## 8.0.011

2023-05-19公開

### Refmac5.8.0411

http://fg.oisin.rc-harwell.ac.uk/scm/loggerhead/refmac/5.8/revision/474

5.8.0405からの変更点

- 中性子関係の機能強化 (dfractionの精密化対象の細かい調整など)
- 結合した2原子が他と一切相互作用してない場合(PDB 4rozのMg-Oなど)に二次微分行列が特異になって最適化が上手くいかなくなる問題の修正
- monomerあたりのtorsion angle restraintsの数が600に制限されていてJSGなどでコケる問題の修正
- `make link no` なのにmetal-ligand linkが追加されてしまう問題の修正

### acedrg 275

271からの変更点

- `LINK: RES-NAME-1 CYS ATOM-NAME-1 SG RES-NAME-2 FAD ATOM-NAME-2 C8M` がコケる問題の修正
- `LINK: RES-NAME-1 MS6 ATOM-NAME-1 C RES-NAME-2 ALA ATOM-NAME-2 N CHANGE BOND C S DOUBLE 1` でsp2_sp2なのにsigma=0になったりする問題の修正
- 同linkで主鎖Nのenergy typeがN31とかN32とかになってしまう問題の修正
- D65やQ0Sなどの平面とtorsion angleに関する問題の修正
- \_chem\_mod\_tor.new\_period が"."だとgemmiが読めないのでdeleteの場合でも値を書くように修正 (gemmi側も修正済み)
- など？

### monomer library

https://github.com/MonomerLibrary/monomers/commits/ccp4-8.0.011

- acedrg 275がアサインする新しい窒素のenergy typeの情報を追加 (ener_lib.cif) [#14](https://github.com/MonomerLibrary/monomers/pull/14)
- 新link: DNA-SER, DNA-TYR [#24](https://github.com/MonomerLibrary/monomers/pull/24), 糖鎖関連 [#2](https://github.com/MonomerLibrary/monomers/pull/2)
- aliasの追加．詳細は[refmacat paper]({{<ref "refmacat_paper.md" >}})参照 [#21](https://github.com/MonomerLibrary/monomers/pull/21)
- 水素原子座標を決めるのに足りてなかった制約情報を補完 (refmacatのデバッグで見つかった) [#17](https://github.com/MonomerLibrary/monomers/pull/17)
- などなど

### LIBG

ここで報告されていたmmCIF読めないエラーがついに修正されました．
https://www.jiscmail.ac.uk/cgi-bin/wa-jisc.exe?A2=ind2211&L=CCPEM&P=R58439

mmCIFのlabel_\*とauth_\*問題絡みです．

### gemmi 0.6.0

https://github.com/project-gemmi/gemmi/releases/tag/v0.6.0

mmCIFの_atom_site.label_asym_idに記号を使っていてOneDepが受け付けてくれない問題の修正など．
[mmCIF自体は記号の使用を許している](https://mmcif.wwpdb.org/dictionaries/mmcif_pdbx_v40.dic/Items/_atom_site.label_asym_id.html)のにOneDepのルールではダメとか勘弁して欲しかった．

### servalcat 0.4.0

https://github.com/keitaroyam/servalcat/releases/tag/v0.4.0

ServalcatがCCP4にこっそり初登場．
refmacatで"make hout no"が無視されてしまうバグ付き
