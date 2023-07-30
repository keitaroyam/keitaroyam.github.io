---
title: "SHELXCのoutlier rejectionについて"
date: 2023-07-29T22:54:53+01:00
math: true
---

昔，ANODEで計算する異常分散差フーリエが，他のプログラム(phenixやRefmac)と比べてもS/Nが良いことが気になり，調査したときのメモを発掘したのでここに清書．
ANODEが良いのはその前処理に使っているSHELXCによるoutlier rejectionが大いに効いていることが分かった．

SHELXCは以下の2つの条件に基づいてrejectionを行う模様．

条件1:
$$
\sigma^2(I^+)>6.25\sigma^2(I^-) {\rm ~or~ } \sigma^2(I^-)>6.25\sigma^2(I^+)
$$
条件2:
$$
\min(\sqrt{I^+}, \sqrt{I^-}) \le 4\sqrt{ (\sqrt{I^+ +\sigma(I^+)} - \sqrt{I^+})^2 + (\sqrt{I^- +\sigma(I^-)} - \sqrt{I^-})^2 }
$$
ただし\\(I < 0\\)のときは\\(I = 0\\)とする．

いかにもSHELXらしい，empiricalな方法という感じ．
このrejection後，SHELXCは\\(e^{-2.5s^2/4}\\)を掛けて出力している．さらにANODEは\\(B=10\\)を掛けて計算しているようだ．

unmerged intensityを与えた場合の挙動は調査しきれていない．昔は重みなしで平均を計算してしまっていたが，ある時点から重み付けするようにはなったらしい(10年前に聞いた話)．
あとは\\(\sigma(\langle I \rangle)\\)はinternal/external varianceの最大値として計算されているようだ．

上記rejectionを確認するために書いたと思われるコードも発掘したので，動くかも分からないがいちおう載せておく．Python2だ…

```py
import iotbx.file_reader
from cctbx.array_family import flex
import math

if __name__ == "__main__":
    f1 = "xscale.sca"
    fano = "anode_fa.hkl"

    i_org = iotbx.file_reader.any_file(f1).file_server.miller_arrays[0]
    i_org.show_summary()
    print
    fa = iotbx.file_reader.any_file(fano+"=amplitudes").file_server.miller_arrays[0]
    fa.show_summary()
    fa = fa.customized_copy(crystal_symmetry=i_org.crystal_symmetry())

    i_org_fa = i_org.anomalous_differences()

    matches = i_org_fa.match_indices(fa)
    flags = flex.bool(i_org.size(), False)
    flags.set_selected(matches.pairs().column(0), True)

    ofs = open("sca_common.dat", "w")
    print >>ofs, "h k l d I sig common"
    for hkl, r, d, s, f in zip(i_org_fa.indices(), i_org_fa.d_spacings().data(), i_org_fa.data(), i_org_fa.sigmas(), flags):
        print >>ofs, "% 4d % 4d % 4d" % hkl,
        print >>ofs, r, d,s,f


    asu, matches = i_org.match_bijvoet_mates()
    ip = i_org.select(matches.pairs().column(0))
    im = i_org.select(matches.pairs().column(1))

    # flag is True when not rejected
    ofs = open("test_reject.dat", "w")
    print >>ofs, "h k l Ip Im SA flag"
    for hkl, Ip, SigIp, Im, SigIm in zip(ip.indices(),
                                         ip.data(), ip.sigmas(),
                                         im.data(), im.sigmas()):
        print >>ofs, "% 4d % 4d % 4d" % hkl,
        print >>ofs, Ip, Im,

        S, T = SigIp**2, SigIm**2
        flag = not (S>6.25*T or T>6.25*S)
        if flag:
            Fp, Fm = math.sqrt(max(Ip, 0)), math.sqrt(max(Im, 0))
            sa = math.sqrt( (math.sqrt(Fp**2+SigIp)-Fp)**2 + (math.sqrt(Fm**2+SigIm)-Fm)**2 )
            flag = min(Fp,Fm) > 4*sa

        print >>ofs, sa, flag
```

当時，かなりがっつり反射数が減ることに驚いたような記憶がある．
適切なweightingを考えるよりも経験的な方法でrejectしてしまう方がいいというのはちょっと残念な感じがするが，もっと良い方法にたどり着けたらいいですね．
