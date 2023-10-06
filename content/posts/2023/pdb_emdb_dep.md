---
title: "PDB & EMDB登録メモ"
date: 2023-07-19
---

最近たくさん登録作業をしたので，以前学生向けに書いたメモをここに再掲しようと思う．
[2021年のPDBj講習会資料](https://pdbj.org/news/workshop20210120?lang=ja)も参照．

登録には，[how to deposit EM MAP](https://www.wwpdb.org/deposition/tutorial#electron-microscopy-volume-map-depositions)にあるように，PDB or mmCIFファイルのほか，以下の情報が必要です．

1. Primary map along with voxel size and recommended contour level
   - 特にルールは無さそう．モデリングに使った主なマップで良いはず．postprocess_masked.mrcなど
2. Image of the map (500 x 500 pixels in .jpg, .png, etc. format)
   - ウェブでサムネイルとして表示される絵．何でも良い．[例](https://www.ebi.ac.uk/pdbe/emdb/release/map/)
3. Half maps (as used for FSC calculation; two maps must be uploaded)
4. Mask
5. FSC curves
   - RELIONの場合はPostProcessで出力されるpostprocess_fsc.xmlをアップできる

決めておくべきこと・調べておくべきこと

* contact author全員のORCiD
  * PDB/EMDBと連絡を取る人+PIのみ．公開されない
* Entry title & authors
  * Surname, F.M.形式で書く
* Funding (organisationとnumber)
* unreleasedに載せるか隠すか
* 分子ごとのmolecule name
* 配列（タグの切れ残り等含め，凍結した試料に含まれた配列全部）
  * モデルと配列が一致していることを確認しておく．食い違っていると進めない
* sourceとexpression host
* 各マップのrecommended contour level (モデルのvalidationにも使われる)
* グリッド作製時のpH, 冷媒，defocus範囲 (nm), 電顕と検出器の型，Average electron dose
* 解析手法, 粒子数, 分解能

## よくあること
* "Still processing. Processing done."と出てるのにページ遷移しないときがある．リロードしてしまって大丈夫そう
* 他のエントリから情報をインポートできるのは最初の画面だけ
* パスワード自動入力が有効だと，Do you want to import information from a previous wwPDB deposition?をNoにしてるのに（ID/PWも空欄にしてるのに），アップロードから先に進むときThe related deposition ID is the same as the current deposition...と出て入力し直しになる現象が起きる．進む前にもう一度確認するようにした方が良い
* 配列アラインメントがあまり賢くなく，gapがあるときに失敗することが多い (annotatorに連絡してsubmitできるようにしてもらう必要がある)

## Validationに関して
### Bond angle outlier
bond angleはsigmaが異様に小さい場合があり，そういう場合は無視して良いと思われる．
例えばARGのNE-CZ-NH1角は，PDBの理想値は120.3±0.5となっているが，[CCP4 monomer library](https://github.com/MonomerLibrary/monomers/blob/0710be469f381e44eeb2b97d21d8c322f24512e1/a/ARG.cif#L164)では120.1±1.5である．このため，例えばモデルの値が117.6だったとき，CCP4的には1.7σの残差だが，PDB的には5.4σも理想値からずれていることになり，怒られる．どちらのsigmaが妥当かは分からないが，0.5°は厳しすぎる．

### ARGがplane outlierになる
[別記事を書きました]({{<ref "arg_torsion.md" >}})

### EM (単粒子)のモデルvalidation
入力したrecommended contour levelに基づいて行われるが，それがそもそもおかしい．
マップ全体を同じlevelで見るものではないし，また同じsharpeningで見るものでもない．同様の理由でQ-scoreを使うのもおかしい．
現状，マップに対する構造のvalidationは見るべきでない．
（もちろん全く違うモデルをアップしてしまってマップに全く入ってない，というようなレベルのミスは見つけられるが）

## 入力欄とmmCIFの対応メモ

| Field      | mmCIF |
| ----------- | ----------- |
| Keywords - classification | _struct_keywords.pdbx_keywords |
| Keywords - keywords | _struct_keywords.text |
| EM sample - sample name | _em_entity_assembly.name |
| EM sample - molecular weight | _em_entity_assembly_molwt.value |
| EM experiment - Point group | _em_single_particle_entity.point_symmetry |

ソースから分かる場合もある．inputタグのdata-tag要素に名前が，divタグのdata-category要素にカテゴリ名が書いてあるようだ

## Assembly
[Kichi氏からの情報](https://twitter.com/kichi_hrs/status/1676975131472830464)で，実はmatrixを一つの欄に複数書いて良いらしい．[資料](https://pdbj.org/news/pssj2023?lang=ja)．

## 改善してほしい点のメモ
* upload失敗したときにpixel size, recommended contour levelが消えてしまう問題
* pixel size入力は任意にしてほしい (ヘッダが正しい時はヘッダのままで良い)
* Assemblyを言葉で書く欄，書くべき内容が不明瞭なので，必須項目でなくて良いように思う．ヘテロ二量体の五量体の場合にdecamerと書く人もいればpentamerと書く人もいるだろうし．明らかにおかしい状態を見つける役には立つだろうけど(penramerと書いてるのにmonomer状態にしか見えないとか)．今は3D表示もついたのでそれで十分では
* mmCIFに_refine.ls_d_res_highがあるとvalidation reportに書かれるauthor-provided resolutionに使われるようだが，これは構造精密化に使った分解能範囲であって，単粒子解析としての分解能は_em_3d_reconstruction.resolutionから取ってほしい
