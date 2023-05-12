---
title: "Cootを使った核酸モデリングのtips"
date: 2023-05-12T19:58:59+01:00
tags: [coot]
---

順次追記予定

## base pair & stacking restraints
CCP4に含まれているプログラムLibGで作製したrestraintをCootに読ませて使うことができる．
LibGの文献は以下を参照
1. Brown et al. "Tools for macromolecular model building and refinement into electron cryo-microscopy reconstructions" [Acta Cryst. (2015). D71, 136-153](https://doi.org/10.1107/S1399004714021683)
2. Kovalevskiy et al. "Overview of refinement procedures within REFMAC5: utilizing data from different sources" [Acta Cryst. (2018). D74, 215-227](https://doi.org/10.1107/S2059798318000979)

注: mmCIFの読み込みバグがCCP4 8.0.011で修正されているので，当バージョンの使用を推奨．

使い方は `libg` と打てば表示される．PDBファイルを使う場合は

```bash
libg -p input.pdb -o libg.txt
```

とするとRefmacのexternal restraintsが記述されたlibg.txtが作られる．
以下は2zm5を使った例 (抜粋):

```
exte dist first chain C resi 23 ins  . atom N6 second chain C resi 12 ins . atom O4 value 2.94 sigma 0.15 type 1 
exte dist first chain C resi 23 ins  . atom N1 second chain C resi 12 ins . atom N3 value 2.84 sigma 0.1 type 1 
exte torsion first chain C resi 23 ins  . atom C2 second chain C resi 23 ins . atom N1 third chain C resi 12 ins . atom N3 fourth chain C resi 12 ins . atom C4 value 180 sigma 15 type 1 
...
exte stac plan 1  firs resi 10 ins . chai C atoms { C1' N9 C8 N7 C5 C6 O6 N1 C2 N2 N3 C4 }  plan 2  firs resi 11 ins . chai C atoms { C1' N1 C2 O2 N3 C4 N4 C5 C6 }  dist 3.4 sddi 0.2  sdan 6.0 type 1
...
```

塩基対の水素結合に対して結合距離制約(exte dist)が，塩基のスタッキングに対して平面間角度と距離の制約(exte stac)が作られる．
記法については[refmac keywords (version 5.5.0026 and later)](https://www2.mrc-lmb.cam.ac.uk/groups/murshudov/content/refmac/refmac_keywords.html)を参照．

このファイルはRefmacに読ませて使えるが(servalcatの場合は `--keyword_file` で指定する)，Cootに読ませる場合は

1. Calculate - Modules - Restraints でRestraintsメニューを出現させる
2. Restraints - Read Refmac Extra Restraintsを選択，上記で作成したファイル(libg.txt)を指定する

上手く読めていれば[このFig. 8d](https://journals.iucr.org/d/issues/2015/01/00/ba5226/ba5226fig8.html)のように表示されるはず．

ちなみにPython scriptingで読ませたい場合は

```py
add_refmac_extra_restraints(imol, "libg.txt")
```

塩基対が崩れすぎている場合，libgが認識してくれない可能性がある．
その場合はIdeal RNA (or DNA)から置き直したほうが早いかも