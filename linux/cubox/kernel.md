# kernel

HDMI/audioを使わないのであれば、公式kernelを使える。

!!! note
    4.19系でもしかしたらHMDI系のサポートが入るかもしれない。
   

## kernel config

ncursesを使った新しい設定画面を使う場合は以下のコマンドで設定を行う。

```shell-session
 $ make nconfig
```

ncursesの古い設定画面の場合は以下のコマンド。

```shell-session
 $ make menuconfig
```


!!! info "arm/arm64系のdeviceに必要なmoduleについて"
    基本的に、arch/arm*/boot/dts/配下のデバイスツリーファイルに記載されているmodule系を有効にすればよい。
	例として、[dove-cubox.dts](https://github.com/torvalds/linux/blob/master/arch/arm/boot/dts/dove-cubox.dts)の中には`"silabs,si5351a-msop"`の記載があり、[livegrep](https://livegrep.com/search/linux)で検索すると
	`Silicon Labs Si5351A/B/C clock generator driver`を有効にすればよいことがわかる。

	
有効にすべきポイントは以下の通り。

```
TODO: 有効にすべきポイントを書く
TODO: sampleのkernel configファイルを晒す
```


## kernel compile

CPU数に応じて`-j2`オプションを増減させる。

```shell-session
 $ make -j2
```


## kernel install

modulesのinstallは通常と変わらず。

```
 $ make modules_install
```

armのkernelはdevice-treeをロードすることで特定の機器のmodule類を初期化することができる。
cuboxの場合はkernelの末尾にdevice-treeを引っ付けてロードする形式がとられていたので踏襲する。

```shell-session
 $ cd arch/arm/boot/
 $ cat zImage dts/dove-cubox.dtb > zImage.cubox
 $ mkimage -A arm -O linux -C none -T kernel -a 0x00008000 -e 0x00008000 -n 'linux-cubox' -d zImage.cubox uImage
```

出来上がったuImageをu-bootで読み出すようコピー、設定する

```shell-session
 # cp uImage /boot/
```

u-bootでロードする`boot.scr`を作成する。
!!! summary "/boot/boot.txt"
    ``` text tab="eSATA"
    setenv bootargs 'console=ttyS0,115200n8 root=/dev/sda3 rootwait rootfstype=ext4'
    ext2load ide 0:1 0x00200000 /uImage
    bootm
    ```
    
    ``` text tab="microSD"
    setenv bootargs 'console=ttyS0,115200n8 root=/dev/mmcblk0p3 rootwait rootfstype=ext4'
    ext2load mmc 0:1 0x00200000 /uImage
    bootm
    ```

`boot.txt` を `boot.scr`に変換する。

```shell-session
 # mkimage -A arm -O linux -T script -C none -n "uBoot commands" -d boot.txt boot.scr
```


## 情報源

- [index : linux-arm.git](http://git.arm.linux.org.uk/cgit/linux-arm.git/)
- [Linux-ARM Kernel](http://archive.armlinux.org.uk/lurker/list/linux-arm-kernel.html)
