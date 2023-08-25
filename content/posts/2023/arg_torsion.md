---
title: "Refmac/Cootのtorsion angle restraintとArg問題"
date: 2023-08-24
math: true
tags: [coot, refinement]
---

## monomer libraryとtorsion angle
CCP4 monomer libraryにはねじれ角(torsion angle)の理想値・周期・標準偏差が記述されています．
例えばARGの定義は https://github.com/MonomerLibrary/monomers/blob/master/a/ARG.cif にありますが，現時点でのtorsion angle restraintsを眺めてみると，以下のようになっています．
ARGの構造と原子名の対応については[RCSB PDB - ARG Ligand Summary Page](https://www.rcsb.org/ligand/arg)でLabelsボタンを押して確認してください．
```star
loop_
_chem_comp_tor.comp_id
_chem_comp_tor.id
_chem_comp_tor.atom_id_1
_chem_comp_tor.atom_id_2
_chem_comp_tor.atom_id_3
_chem_comp_tor.atom_id_4
_chem_comp_tor.value_angle
_chem_comp_tor.value_angle_esd
_chem_comp_tor.period
ARG chi1 N CA CB CG -60.000 10.0 3
ARG chi2 CA CB CG CD 180.000 10.0 3
ARG chi3 CB CG CD NE -60.000 10.0 3
ARG chi4 CG CD NE CZ 180.000 10.0 6
ARG chi5 CD NE CZ NH2 180.000 5.0 2
ARG hh1 NE CZ NH1 HH12 180.000 5.0 2
ARG hh2 NE CZ NH2 HH22 0.000 5.0 2
ARG sp3_sp3_1 C CA N H 180.000 10.0 3
ARG sp2_sp3_1 O C CA N 0.000 10.0 6
```
右側3つの数字がvalue_angle, value_angle_esd, periodで，理想値・その標準偏差・周期を表しています．
例えばchi1は理想値-60±10°ですが，周期が3なので360/3=120°おきに理想値が存在する，つまり-60, 60, 180°は全て理想値ということになります．

chi5はNE-CZ結合周りのねじれ角ですが，標準偏差が5°と他よりも小さく設定されています．これはAceDRGのデフォルトで，sp2-sp2 (つまり二重結合性の)結合周りは強く制約するために小さくしておこうということです．
以前はこういう状況では平面の制約を使っていたと思いますが，torsion angle restraintに置き換わりました．
平面は本当に厳密に平面になる場合（ベンゼン環など）に限定して使う思想になっています．

[Moriarty et al (2020)](https://doi.org/10.1107/S2059798320013534)でも解説されているように，Argの{CD, NE, CZ, NH1, NH2} "平面"からCD原子は少し飛び出しやすいことが知られています．
大きく外れるのはおかしいですが，厳密に同一平面に乗るように制約するのもおかしい．
PHENIXはあくまで平面のままで，ただしCD原子だけ少し平面から飛び出すことを多めに許容する仕組みを導入しています．
一方，CCP4は平面ではなくtorsion angleを使うべきという思想で，平面はあくまで{NE, CZ, NH1, NH2}だけ定義し，CD原子についてはchi5で制約しています．

## Servalcat/Refmacにおける制約

さてServalcat/REFMAC5では，以下の式でねじれ角の制約を導入します．
$$
\frac{w^2}{2\sigma^2} (\theta_{\rm model} - \theta_{\rm ideal})^2
$$
ふつうの最小二乗の形です．残差を計算する際，もちろん周期性は考慮されて，現在の角度から最も近いところにある理想値が使われます．
式にσが入ってることからも分かるように，小さなσの場合はより強く制約するというわけです．
ただし，monomer libraryにある全てのねじれ角を使うわけではなく，sp2_sp2\*と，ペプチドのomega，Proを除くアミノ酸のchi*だけが使われます．

## Cootにおける制約

次にCootですが，ある時からCootは個別のσの値を使わなくなってしまいました．Cootのtorsion angle restraintは以下の式で行っているようです(nは周期)．
$$
{\rm torsion\\_restraint\\_weight} \times 11 \times \frac{1-\cos{n (\theta\_{\rm model} - \theta\_{\rm ideal})}}{2}
$$

[ソースコード上](https://github.com/pemsley/coot/blob/58ccfeebc5f76e5e4373269c1fa510dec22e3a8d/ideal/distortion.cc#L1461)では個別のtorsion_restraint_weightを設定できるようになっていますが，これは実質全体で1つの値であり，set_torsion_restraints_weight()から変更できます．右端R/RCボタンからMore controlを出して，その中のTorsions weightからも調整できます．

CootはRefmacと違って最小二乗ではありませんが，Δθが微小量の場合にTaylor展開すると
$$
\sigma^2=\frac{4}{11 \times {\rm weight}}
$$
となるのでデフォルトのweight = 1はσ = 34.6°に対応し．weight = 12でσ = 10°，weight = 48でσ = 5°相当ということになります．

Cootではtorsion angle restraintsはデフォルトでOFFです．
また，ONにしてもweight = 1ではかなりゆるい事になるので，実質あまり機能しないと思われます．
これが実際かなり問題であるわけです．二重結合性のねじれ角は強めに制約しないといけないのに，それができません．
weightをきつくしてしまうと，他のねじれ角まで強くなってしまいます．大問題です！

また，CootはRefmacのようにねじれ角の取捨選択をほとんど行いません．
[Proは除外](https://github.com/pemsley/coot/blob/58ccfeebc5f76e5e4373269c1fa510dec22e3a8d/ideal/make-restraints.cc#L1091)するようですが，他はそのまま使ってしまいます．
monomer libraryでは核酸のリボース環は2'-endoと3'-endoの両方について定義されていて，これを同時に使おうとすると当然おかしくなります．
こういった理由からも，Cootのtorsion angle restraintsをONにすることは推奨しません．本当に必要なときだけ，weightを大きくして使うべきです．

じゃあ二重結合性の平面(アミドなども含む)をちゃんと維持させるにはどうしたら良いのか．
残念ながらCootが直るまでは，平面制約を加えたdictionary cif fileを自分で用意して使うしか無いと思います．
ARGの場合，平面は
```star
loop_
_chem_comp_plane_atom.comp_id
_chem_comp_plane_atom.plane_id
_chem_comp_plane_atom.atom_id
_chem_comp_plane_atom.dist_esd
ARG plan-1 C 0.020
ARG plan-1 CA 0.020
ARG plan-1 O 0.020
ARG plan-1 OXT 0.020
ARG plan-2 CD 0.020
ARG plan-2 CZ 0.020
ARG plan-2 HE 0.020
ARG plan-2 NE 0.020
ARG plan-3 CZ 0.020
ARG plan-3 NE 0.020
ARG plan-3 NH1 0.020
ARG plan-3 NH2 0.020
ARG plan-4 CZ 0.020
ARG plan-4 HH11 0.020
ARG plan-4 HH12 0.020
ARG plan-4 NH1 0.020
ARG plan-5 CZ 0.020
ARG plan-5 HH21 0.020
ARG plan-5 HH22 0.020
ARG plan-5 NH2 0.020
```
と定義されていますが，ここのplan-3にCD原子を加える，つまり

```star
ARG plan-3 CD 0.020
ARG plan-3 CZ 0.020
ARG plan-3 NE 0.020
ARG plan-3 NH1 0.020
ARG plan-3 NH2 0.020
```
としてやります．ただしこれだと制約が強すぎるので，CD原子が同一平面にほぼ来てしまいます．特に高分解能の精密化をしてるときは気をつけてください．そのあとでRefmac等を流せば緩和されるとは思いますが．

## PDB validationにおけるArgの平面問題

Coot/RefmacでrefineしたモデルをPDB登録したら，やたらARGの平面が悪いと怒られた経験があるかも知れません．
どうやらPDB validationは，{CD, NE, CZ, NH1, NH2}の平面のrmsdを評価し，それが約0.07 Å以上あるとoutlierとみなすようです．
上記で紹介した文献にもあるように，そもそもこれは平面ではないので，間違ったvalidationです．
ただし，平面のrmsdに対して0.07 Åという基準は，わりと甘めではあります．厳密な関係はありませんが，chi5のdeviationで言うと約15°に相当します．
現状のchi5のsigma (5°)から考えると15°というのは3σなので，3σ程度のdeviationはまあまあ起きるだろうという感じですが，そもそもこのsigmaの値は適切なのだろうかという疑問はあります．
低分子の統計では標準偏差は5°より大きいようですが，高分子の精密化ではもっと小さくしたほうが良いかも知れません．グループ内でも議論してる途中です．