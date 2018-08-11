# lftp

## LFTPとは

主にFTP転送、ミラーリングで使われる転送プログラム。

!!! quote "公式サイトより引用"
    LFTPは専れされたファイル転送プログラムで、いくつかのプロトコル(ftp/http/sftp/fish/torrent)をサポートしている。
    BASHのように、jobコントロール機能を持ち、入力のためにreadlineライブラリを使っている。
    ブックマークやbuilt-in mirrorコマンドを持ち、幾つかのファイルを並行して転送することも可能。
	LFTPは信頼性を考慮して設計されている。

!!! info "最新バージョン"
    公式サイトで確認できるのは4.8.4(2018/08/01 Release)


## 使い方

基本は対話型プログラム。対話を終了するには`exit`コマンドか、Ctrl+Dを入力。

### 接続

```shell-session
 $ lftp -u username,password ftp.example.com
```

### built-in commands

接続後はbuilt-inコマンドを使って情報の取得・転送を行う。
以下はその一例。

#### ls

リモートファイル一覧の取得

```
lftp username@ftp.example.com:~> ls
-rw-r--r--    1 53         bind              490 Aug 11 10:39 index.html
```

#### cat

リモートファイルを標準出力表示

```
lftp username@ftp.example.com:~> cat index.html
<!DOCTYPE HTML>
<html lang="ja">
<html>
<head>
<meta charset="utf-8">
</head>
<body>
</body>
</html>
```

#### mirror

ローカル、リモート間のミラーリング

``` tab="remote->local"
lftp username@ftp.example.com:~> mirror / ~/dir
```

``` tab="local->remote"
lftp username@ftp.example.com:~> mirror -R /dir /
```

### 対話せずに実行

sshのように、`-e`オプションにて実行するbuilt-inコマンドを指定することが可能。

```shell-session
 $ lftp -u username,password -e "mirror / ~/localrepo; quit" ftp.example.com
```

## 情報源

- [LFTP - Official](https://lftp.tech)
- [LFTP - Wikipedia(ja)](https://ja.wikipedia.org/wiki/Lftp)
