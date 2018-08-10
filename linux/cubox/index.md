# linux on cubox(1st gen)

公式Wikiから消え去ってしまったcuboxの第1世代上でLinuxを動かすための情報。

## 現状

昔はSolidRunの開発者がlinux kernelのdriverを書いたリポジトリがあり、そのkernelを使っていた。
更新は止まってしまったが、公式kernelでのサポートが進み、現在はHDMI/audio以外は動いている。

HDMI/audioについても4.19向けにパッチが投げられており、適用されれば使えるようになる、が、
"mervel,dove-framebuffer"に該当するドキュメントはどこにあるの？というレビューから先の
返信が無いため、取り込まれるか微妙。

## 情報源

- [armada5xx (1st gen. cubox)](http://forum.solid-run.com/viewforum.php?f=25&sid=8984fe627cd6e883ba4c074959b8e25a)
- [arm linux mailing list](http://archive.armlinux.org.uk/lurker/list/linux-arm-kernel.html)
