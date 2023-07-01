---
title: "Cootを使った核酸モデリングのtips"
date: 2023-05-12T19:58:59+01:00
tags: [coot]
ShowToc: true
---

順次追記予定

---
注: 現在最新版のCoot 0.9.8.8および0.9.8.7 (CCP4 8.0.008-12)は核酸-タンパク複合体の精密化に深刻なバグあり．0.9.8.6は核酸単体でもダメです．核酸を含む精密化をやるときは0.9.8.5を使ってください（ただしこのバージョンは糖鎖がダメです）．

あるいは以下の方法で回避可能です．[こちら]({{<ref "ccp4_8_updates.md#coot-0988" >}})で解説したようにAA-RNAリンクが悪さをしているので， `$CLIBD_MON/list/mon_lib_list.cif` (CCP4パッケージに含まれるmonomer libraryを使用している場合) の中の
```
AA-RNA . DEL-OXT peptide . DEL_HO3p DNA/RNA aminoacyl-RNA
```
を
```
#AA-RNA . DEL-OXT peptide . DEL_HO3p DNA/RNA aminoacyl-RNA
```
という感じでコメントアウトすれば使えるようになります．ただしアミノアシルtRNAの結合が作れなくなるので注意してください．

---

## base pair & stacking restraints
### LIBGを使う
CCP4に含まれているプログラムLIBGで作製したrestraintをCootに読ませて使うことができる．
LIBGの文献は以下を参照
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

塩基対が崩れすぎている場合，LIBGが認識してくれない可能性がある．
その場合は以下の方法で手動で設定できるが，Ideal RNA (or DNA)から置き直したほうが早いかも

### 手動で設定する

塩基対は，

1. Calculate - Modules - Restraints でRestraintsメニューを出現させる
2. Restraints - RNA A form bond restraints または DNA B form bond restraints を選択
3. 塩基対を組ませたい残基を1つずつクリック (計2回)

で設定できる．紛らわしい名前をしているが，[ソース](https://github.com/pemsley/coot/blob/8711a215318fdd934ddc745cfbe56a83e31362c0/python/user_define_restraints.py#L159)を読む限り，塩基の原子間距離の制約を設定しているだけなので，A formやB formになるわけではない（主鎖の制約ではない）．

スタッキングは，

1. Restraints - Add Parallel Planes Restraintを選択
2. スタッキングさせたい残基を1つずつクリック (計2回)

で塩基平面の平行性を保つ制約が設定できる．実装は上記ソースから `user_defined_add_planes_restraint()` を参照．

この方法だとDNA-RNA heteroduplexの塩基対を組ませることができない．この機能自体変えてもらうほうが良いように思うが，とりあえず以下のスクリプトを適当な名前(.py)で保存してCalculate - Run Scriptから実行すればRestraintsメニューに"base pair restraints"が出現するので，これで組めるようになるはず．

```py
# original: https://github.com/pemsley/coot/blob/refinement/python/user_define_restraints.py

def add_base_restraint(imol, spec_1, spec_2, atom_name_1, atom_name_2, dist):
    add_extra_bond_restraint(imol,
                           spec_1[2],
                           spec_1[3],
                           spec_1[4],
                           atom_name_1,
                           spec_1[6],
                           spec_2[2],
                           spec_2[3],
                           spec_2[4],
                           atom_name_2,
                           spec_2[6],
                           dist, 0.035)
def a_u_restraints(spec_1, spec_2):
    imol = spec_1[1]
    add_base_restraint(imol, spec_1, spec_2, " N6 ", " O4 ", 3.12)
    add_base_restraint(imol, spec_1, spec_2, " N1 ", " N3 ", 3.05)
    add_base_restraint(imol, spec_1, spec_2, " C2 ", " O2 ", 3.90)
    add_base_restraint(imol, spec_1, spec_2, " N3 ", " O2 ", 5.12)
    add_base_restraint(imol, spec_1, spec_2, " C6 ", " O4 ", 3.92)
    add_base_restraint(imol, spec_1, spec_2, " C4 ", " C6 ", 8.38)
def g_c_restraints(spec_1, spec_2):
    imol = spec_1[1]
    add_base_restraint(imol, spec_1, spec_2, " N6 ", " O4 ", 3.12)
    add_base_restraint(imol, spec_1, spec_2, " N1 ", " N3 ", 3.04)
    add_base_restraint(imol, spec_1, spec_2, " N2 ", " O2 ", 3.14)
    add_base_restraint(imol, spec_1, spec_2, " C4 ", " N1 ", 7.73)
    add_base_restraint(imol, spec_1, spec_2, " C5 ", " C5 ", 7.21)
def user_defined_base_pair():
    res_name_from_atom_spec = lambda x: residue_name(*x[1:5])
    def make_restr(*args):
        spec_1 = args[0]
        spec_2 = args[1]
        res_name_1 = res_name_from_atom_spec(spec_1)
        res_name_2 = res_name_from_atom_spec(spec_2)
        print("DEBUG:: have resnames", res_name_1, res_name_2)
        if (res_name_1 in ("G", "DG") and res_name_2 in ("C", "DC")):
            g_c_restraints(spec_1, spec_2)
        if (res_name_1 in ("C", "DC") and res_name_2 in ("G", "DG")):
            g_c_restraints(spec_2, spec_1)
        if (res_name_1 in ("A", "DA") and res_name_2 in ("U", "DT")):
            a_u_restraints(spec_1, spec_2)
        if (res_name_1 in ("U", "DT") and res_name_2 in ("A", "DA")):
            a_u_restraints(spec_2, spec_1)
    user_defined_click_py(2, make_restr)

menu = coot_menubar_menu("Restraints")
add_simple_coot_menu_menuitem(menu, "base pair restraints...", lambda func: user_defined_base_pair())
```

## 二重らせんのフィッティング

1. モデルを置きたい場所の中心あたりに移動
2. Calculate - Other Modelling Tools - DNA & RNA modelsを選択
3. RNA/DNA, A/B form, double strandedを選択し配列を入力
4. モデルが出現するので，Morph - Jiggle-fit This Molecule (Simple)を実行．マップにはまればOK．はまらなければ何度か繰り返す．ダメなときはRotate Translate Zoneで場所を動かす（回転は全探索されるので問題ない）

(Morphメニューが無いときはFile - CurlewからMorphをインストールする．Refineメニューもこのあとで使うので，無い場合はChain Refineもインストールする)

大まかにモデルが合ったら，精密化する．分解能が十分良いな普通にすれば問題ないが，分解能が悪いときは以下の方法で行う方が良い．

まず構造を極力維持させるために，self-restraintsを設定する

1. Calculate - Modules - Restraints でRestraintsメニューを出現させる
2. Restraints - Generate All-Molecule Self Restraints 4.3

もし二重らせんモデルを既存のモデルにマージした上で行いたい場合は，マージ後，Calculate - Scripting - Pythonから

```py
chains = ["A", "B"]
imol = 0
all_res = sum([residues_in_chain(imol, x) for x in chains], [])
generate_local_self_restraints_by_residues(imol, all_res, 4.3)
```

とする．chainsのIDは二重らせんのものに，imolはその分子番号(Display Managerから確認)に置き換えること．

次に実際に精密化．特に電顕SPAのマップのときは，最初にWeightを適切に調整すること．
右上のR/RCボタンからRefinement Weight横のEstimateボタンを押し，実際に適当な場所でsphere refine等を行ってから"Bonds:"の値が0.5-1.0程度に収まるところを探す（この精密化はキャンセルすること）．
また，self-restraintsのGeman-McClure alpha (More Controlにあり)はデフォルトが0.01だが，R/RCを開くまでは1になっている罠があるので注意．

モデルをマージしていなければ，Refine - All-atom Refineする．マージ済みで二重らせんだけrefineしたいときは，Calculate - Scripting - Pythonから

```py
refine_residues(imol, all_res)
```

とする（上記のscriptでall_resを設定済みの前提．必要なときは上のscriptの1-3行目をもう一度実行する）．
もしモデルが変形しすぎるときは上記Geman-McClure alphaを小さくする．逆に，もう少し変形して欲しいときは大きくする．

### 二重らせんモデルの理想化

上記の方法で置くCootの二重らせんモデルは，実はあまり理想的な状態ではない（なお，以前はIdeal RNAみたいな名前だったが，それを指摘したらIdealの名前だけ削られてしまった）．
モデルをもう少し理想化してから精密化することも可能である．
要望があり次第書きます．

## フラグメントのフィッティング

RNA FRABASE http://rnafrabase.cs.put.poznan.pl/ で配列や二次構造でフラグメントの3次元座標が検索できる．低分解能のマップに結構ぴったりはまってくれたりして嬉しい
