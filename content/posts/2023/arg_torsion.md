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

次にCootですが，ある時からCootは個別のσの値を使わなくなってしまいました．Torsion angle restraintをONにしたとき，信号色と一緒に表示されるscoreは以下の式で計算されます([ver 0.9.8.92時点](https://github.com/pemsley/coot/blob/d0ff45f118b2d1f519c0ba748a53e9725f883ed3/ideal/distortion.cc#L1465))
$$
\sqrt{\frac{1}{N}\sum\_j^N 11 w \frac{1-\cos{n\_j (\theta\_{{\rm model},j} - \theta\_{{\rm ideal},j})}}{2}}
$$
ここで$n$は周期，$w$は内部変数torsion\_restraint\_weightで，ソースコード上では個別の値を設定できるようになっていますが，実際は全体で1つの値であり，set\_torsion\_restraints\_weight()から変更できます．右端R/RCボタンからMore controlを出して，その中のTorsions weightからも調整できます．


ただし，なぜか[gradient計算のとき](https://github.com/pemsley/coot/blob/d0ff45f118b2d1f519c0ba748a53e9725f883ed3/ideal/gradients.cc#L1285)は11倍されず，以下の式で寄与が計算されます．
$$
w \frac{1-\cos{n\_j (\theta\_{{\rm model},j} - \theta\_{{\rm ideal},j})}}{2}
$$
ここで，Δθが微小量の場合にTaylor展開して最小二乗の形($w=1$)と比較すると，
$$
\frac{(\Delta \theta)^2}{2\sigma^2} \approx w \frac{(n\Delta \theta\frac{\pi}{180})^2}{4}
$$
となるので，換算式は
$$
\sigma = \frac{1}{n} \sqrt{\frac{6565.613}{w}}
$$
または
$$
w = \frac{6565.613}{\sigma^2 n^2}
$$
となります．周期nによって強さが変わってしまうのがややこしいですね．sp2\_sp2の場合は$n = 2$なので，σ = 5°相当にするにはw = 65ということになります．

Cootではtorsion angle restraintsはデフォルトでOFFです．
また，ONにしてもweight = 1では比較的ゆるい事になるので，実質あまり機能しないと思われます．
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

別解としては，sigmaの小さいtorsion angle以外を消してしまう手も考えられます．
Edit - Restraintsから残基名を選び，Torsionsから必要ないものを消します．
毎回これをやるのも大変なので，そういったcif fileを用意して読ませるほうが楽かもしれません．

## PDB validationにおけるArgの平面問題

Coot/RefmacでrefineしたモデルをPDB登録したら，やたらARGの平面が悪いと怒られた経験があるかも知れません．
どうやらPDB validationは，{CD, NE, CZ, NH1, NH2}の平面のrmsdを評価し，それが約0.07 Å以上あるとoutlierとみなすようです．
上記で紹介した文献にもあるように，そもそもこれは平面ではないので，間違ったvalidationです．
ただし，平面のrmsdに対して0.07 Åという基準は，わりと甘めではあります．厳密な関係はありませんが，chi5のdeviationで言うと約15°に相当します．
現状のchi5のsigma (5°)から考えると15°というのは3σなので，3σ程度のdeviationはまあまあ起きるだろうという感じですが，そもそもこのsigmaの値は適切なのだろうかという疑問はあります．
低分子の統計では標準偏差は5°より大きいようですが，高分子の精密化ではもっと小さくしたほうが良いかも知れません．グループ内でも議論してる途中です．

## torsion angle restraintsの是非と難しさ

二重結合のtorsion angleのようなほぼ確かである場合を除いて，torsion angleに関するrestraintを導入するのはやや危険な行為です．
そもそも構造解析は，torsion angleつまりコンフォメーションを実験的に決定することが目的の一つとも言えます．
Ramachandran restraintやrotamer restraintもtorsion angle restraintの一種ですが，outlierが出ないように強く制約するのは非常に邪悪な行為です．
残念ながら，PDBの最近の（特にSPA）構造の多くはすでに汚染されてしまっています．

二重結合の場合の理想値はほぼ確かであると言いましたが，ここにも少し難しさはあります．エチレンのような単純な場合ならともかく，分子中の二重結合または二重結合性を持つ結合は，周囲の状況によって少し歪むこともよくあります．
Argのchi5はまさにその一つです．化学に基づいて適切に理想値・標準偏差を設定することが望ましいですが，現状AceDRGでそういうことはできていません．