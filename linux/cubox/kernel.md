# kernel

HDMI/audioを使わないのであれば、公式kernelを使える。

!!! note
    4.19系でもしかしたらHMDI系のサポートが入るかもしれない。
    
## kernel compile & install

基本的な部分は同じ。

```shell-session
 $ make nconfig
 $ make
 $ make modules_install
```

armのkernelはdevice-treeをロードすることで特定の機器のmodule類を初期化することができる。
cuboxの場合はkernelの末尾にdevice-treeを引っ付けてロードする形式がとられていたので踏襲する。

```shell-session
 $ arch/arm/boot/
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