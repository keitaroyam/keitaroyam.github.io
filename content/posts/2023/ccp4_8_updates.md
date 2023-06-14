---
title: "CCP4 8.0アップデートまとめ"
date: 2023-06-14
tags: [ccp4]
---

私の興味範囲のみのまとめです．
公式情報: https://www.ccp4.ac.uk/?page_id=3088

リリースノートに記載されてないアップデートも実際にはあるのがCCP4 update.
実際にどのファイルが変わったのかは $CCP4/restore/update.log に記録されています．

## 8.0.012
2023-06-14?公開

### Refmac5.8.0415

http://fg.oisin.rc-harwell.ac.uk/scm/loggerhead/refmac/5.8/revision/478

1. van der Waals, ion半径の調節が可能に
2. refmacatの場合にexternal restraintsのchain IDが正しくinstructionから読めていなかった問題の修正 (exte nonb, exte harm)
3. exte harm chai.. の指定が機能していなかった(exte harm atin chai...とする必要があった)ので修正
<!--4. 一部の水素だけを精密化したときにcalc_flagが全部"."になってしまっていた問題の修正-->

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