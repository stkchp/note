# U-Boot

公式系のWikiの中で拾えるものを拾ってメモしておく。

## 実際にU-Bootは何をするのか

kernelをロードするブートローダ。どのデバイスにOSがinstallされているのか検索してkernelをロードする。

ext2/ext3やFAT、NFS、TFTPをサポートする。
CuBoxは内部フラッシュやmicroSD、eSATA、上のUSBポート、ネットワークかシリアル経由でのbootが可能。

!!!caution "GPTについて"
    デフォルトのブートローダーはGPTを理解することができない。

## CuBoxのブートプロセス

!!!quote "`archive.org`から拾ってきたWikiの和訳"
    ブートプロセスの詳細は以下の通り。
	
    1. 電源ONまたはシステムリセット
    2. メインプロセッサは、内部ユニットとすべてのオンボード周辺機器のHWリセットを実施
	3. メインプロセッサは、チップ内部のROMファームウェアを実行(この工程のチップは上書き不可)
	4. ROMファームウェアはバックパネルにある小さいリセットボタンの状態を読み取る
	    1. 押されている場合、X-modemプロトコル(115200bps 8N1)で送られてきたイメージからbootする
		2. 押されていない場合、SPIフラッシュからイメージをロードしてbootする
	
	X-modemを使ったブートプロセスを使えば、たとえSPIフラッシュがイカれていても、CuBoxをリカバーできる


上記の中の「イメージ」と呼ばれるものが"U-Boot"にあたる。
	
手順さえ分かっていればリカバリー可能。文鎮化は避けられる。
実際に実施した方の記事に以下がある。

- [SolidRun CuBox を使ってみる（その7） | 真・死して屍拾う者無し](http://www.thx-design.net/blog/?p=8283)

## 公式U-Bootについて

公式U-Bootへパッチを投げる作業をSolidRun社員がやっていたが、以下の理由から頓挫している。

- Marvell Dove系のデバイスが少なく、メインライン側のやる気が出ない

Mervellの

## SolidRun社公開イメージについて

以下のInternet Archiveサイトから拾うことが可能。

- [CuBox U-Boot - Internet Archive](https://web.archive.org/web/20140109174216/http://download.solid-run.com:80/pub/solidrun/cubox/u-boot/2009.08_5.4.4_SR1/)


### バイナリバージョン

自分のCuBoxに入っているU-Bootのバイナリを見ると、以下のバージョン。

```
U-Boot 2009.08 (Jan 15 2012 - 00:43:32) Marvell version: 5.4.4 NQ
```

archive.orgに落ちているU-BootのImageは以下。

```
U-Boot 2009.08-dirty (Aug 06 2013 - 19:36:08) Marvell version: 5.4.4 NQ SR1
```

### ファイル情報

`2013/08/06`公開のimageについての情報は以下の通り。

!!!info "md5sum"
    ```
    6b19fbc89b1a8ac452980c9f798eeab1  u-boot-cubox_hynix_cubox_1GB_uart.bin
    40a2cc3ff5af6207d6eddfb42bc8c63c  u-boot-cubox_hynix_cubox_2GB_uart.bin
    b5dc1ffe9bea1edb8274a87711139191  u-boot-cubox_hynix_cubox_unified_spi.bin
    ```

!!!info "ls -l"
    ```
	-rw-r--r-- 1 stkchp stkchp 339464 u-boot-cubox_hynix_cubox_1GB_uart.bin
    -rw-r--r-- 1 stkchp stkchp 339464 u-boot-cubox_hynix_cubox_2GB_uart.bin
    -rw-r--r-- 1 stkchp stkchp 340488 u-boot-cubox_hynix_cubox_unified_spi.bin
    ```	

## U-Bootを自前でビルド

### 公式より引用

ソースコードは[https://github.com/rabeeh/u-boot](https://github.com/rabeeh/u-boot)を利用する。

以下の手順でビルドする。

```shell-session
 $ export CROSS_COMPILE=<path to your favorite ARM cross compiler>
 $ export ARCH=arm
 $ make cubox_config BOOTROM=1 SPIBOOT=1 DRAM=HYNIX_CUBOX NAND=0
 $ make
```

3つ目の手順のところで以下の出力を得るはず。

```
Configuring for mv_dove board...
 * Configured for 88AP510
 * Configured for CuBox
 * HYNIX DDR-3 CuBox Support
 * With USB support
 * Boot from SPI support
 * BOOTROM support
```

以下の2つのバイナリを得ることができる。

- u-boot-cubox_hynix_cubox_spi.bin (SPIフラッシュ書き込み用)
- u-boot-cubox_hynix_cubox_uart.bin (SPIフラッシュがイカれたとき用)

### やってみる

#### クロスコンパイラ

2013年頃はgccは4系だったので、それを使ってコンパイルする。ホストは`x86_64`。
以下のURL先のバイナリを使用(`gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz`)

- [Linaro Releases](https://releases.linaro.org/components/toolchain/binaries/4.9-2017.01/arm-linux-gnueabihf/)

新しいバージョンのgccでも、`-std=gnu89`をCFLAGSに追加してやればコンパイル可能なはず。


パスを通しておく。

#### 手順通りにやってみる

##### config

```shell-session
 $ export CROSS_COMPILE=arm-linux-gnueabihf-
 $ export ARCH=arm
 $ make cubox_config BOOTROM=1 SPIBOOT=1 DRAM=HYNIX_CUBOX NAND=0
```

```
Configuring for mv_dove board...
  * Configured for 88AP510
  * Configured for CuBox
  * HYNIX DDR-3 CuBox Support
  * With USB support
  * Boot from SPI support
  * BOOTROM support
```

手順通りの出力が得られる。

##### make

```shell-session
 $ make
```

特にエラーなし。手順と違い、以下のファイルが得られる。

- u-boot-cubox_hynix_cubox_1GB_uart.bin (CuBox用)
- u-boot-cubox_hynix_cubox_2GB_uart.bin (CuBox Pro用)
- u-boot-cubox_hynix_cubox_unified_spi.bin (CuBox/CuBox Pro両用)

##### 出力例

コンパイル日時等が入るので、md5sumの時刻が変わってくる。

```
U-Boot 2009.08 (Aug 15 2018 - 00:52:02) Marvell version: 5.4.4 NQ SR1
```

!!!info "md5sum"
    ```
    a6995eefefd8c8eef8332fea24099823  u-boot-cubox_hynix_cubox_1GB_uart.bin
    0ce0752e466adcc38a4186169944883c  u-boot-cubox_hynix_cubox_2GB_uart.bin
    23c9be876277d4f50209a0ae2acbdd4c  u-boot-cubox_hynix_cubox_unified_spi.bin
    ```

!!!info "ls -l"
    ```
    -rw-r--r-- 1 stkchp stkchp 339392 u-boot-cubox_hynix_cubox_1GB_uart.bin
    -rw-r--r-- 1 stkchp stkchp 339392 u-boot-cubox_hynix_cubox_2GB_uart.bin
    -rw-r--r-- 1 stkchp stkchp 340416 u-boot-cubox_hynix_cubox_unified_spi.bin
    ```

## SPIフラッシュへの書き込み方

以下はFATでフォーマットされたmicroSDに入っているイメージを書き込む例。公式Wikiのarchiveをほぼコピー。

!!!summary "microSD -> memory"
    ```
    set loadaddr 0x02000000
    mmcinfo
    fatload mmc 0:1 $loadaddr /u-boot-spi.bin
    ```

!!!summary "memory -> internal flash"
    ```
    set size 0xc0000
    sf protect off
    sf erase 0 $size
    sf erase $size 0x10000
    sf write $loadaddr 0 $filesize
    sf protect on
    ```
	
	!!!caution "注意"
	    環境変数を保持したい場合は、2つ目の`erase`コマンドは実行しないこと。
		環境変数はU-Bootのプロンプト上で`printenv`することで表示が可能。
		環境変数を`setenv`で変更した後は`saveenv`SPIフラッシュに保存することが可能。
		
## SPIフラッシュへの書き込み方(Linux)

Linux上だと`/dev/mtd0`にSPIフラッシュが見えている。`mtd-utils`を使って書き込み等が可能。

!!!summary "SPIフラッシュ 情報表示"
    ```shell-session
    # mtdinfo /dev/mtd0
    mtd0
    Name:                           spi0.0
    Type:                           nor
    Eraseblock size:                65536 bytes, 64.0 KiB
    Amount of eraseblocks:          64 (4194304 bytes, 4.0 MiB)
    Minimum input/output unit size: 1 byte
    Sub-page size:                  1 byte
    Character device major/minor:   90:0
    Bad blocks are allowed:         false
    Device is writable:             true
    ```

!!!summary "SPIフラッシュ 全消去"
    ```shell-session
     # flash_erase /dev/mtd0 0 0
    ```

!!!summary "SPIフラッシュ U-Bootイメージの書き込み"
    ```shell-session
     # flashcp u-boot-cubox_hynix_cubox_unified_spi.bin /dev/mtd0
    ```
	
!!!summary "SPIフラッシュ バックアップ"
    ```shell-session
     # dd if=/dev/mtd0ro of=spi-backup-`date +%Y%m%d%H%M`.bin bs=1024 count=832
    ```

## 情報源

- [U-Boot - SolidRun CuBox Wiki - Internet Archive](https://web.archive.org/web/20150103192104/http://www.solid-run.com:80/archive/mw/UBoot)
