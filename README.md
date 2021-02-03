# [ラズベリー・パイで作るホーム・メディア・サーバ](https://git.bokunimo.com/nas/)

本ファイルは、Webページでも閲覧できます：  
[https://git.bokunimo.com/nas/](https://git.bokunimo.com/nas/)

## ラズベリー・パイの準備

### Raspberry Pi OS のインストール

- Raspberry Pi Imager（PC用ソフト）：  
	https://www.raspberrypi.org/downloads/

### 外付けSSDを指定ディレクトリにマウントする

1. ドライブとパーティションを表示する  

	pi@raspberrypi:~ $ `ls /dev/sd*` ⏎  
	/dev/sda  /dev/sda1  

2. パーティションの情報を確認する

	pi@raspberrypi:~ $ `sudo fdisk -l /dev/sda` ⏎  
	Device     Boot Start       End   Sectors   Size Id Type  
	/dev/sda1        2048 250069678 250067631 119.2G 83 Linux  

3. ディレクトリの作成とマウントを実行

	pi@raspberrypi:~ $ `sudo mkdir -pm775 /mnt/ssd1` ⏎  
	pi@raspberrypi:~ $ `sudo mount /dev/sda1 /mnt/ssd1` ⏎  

4. マウントできたかどうかを確認する

	pi@raspberrypi:~ $ `df` ⏎  
	/dev/sda1        123071848   63156 116757004    1% /mnt/ssd1  

5. ファイル共有サーバSambaをインストールする  

	pi@raspberrypi:~ $ `sudo apt-get update` ⏎  
	pi@raspberrypi:~ $ `sudo apt-get install samba` ⏎  

6. 設定ファイルに[nas]を追加する  

	pi@raspberrypi:~ $ `sudo mousepad /etc/samba/smb.conf` ⏎  

	```smb.conf
	[nas]
	comment = NAS for DMS
	path = /mnt/ssd1
	guest ok = yes
	read only = yes
	```

### 外付けSDDの自動マウント先の設定

1. パーティションのリストを確認  

	pi@raspberrypi:~ $ `mount|grep ^/dev/` ⏎   
	/dev/mmcblk0p7　on　/　type　ext4 (rw,noatime)   
	/dev/mmcblk0p6　on　/boot　type　vfat(rw,relatime,～)   
	/dev/sda1　on　/mnt/ssd1　type　ext2 (rw,relatime)  

2. 自動マウントしたいパーティション情報

	マウント先	/mnt/ssd1  
	フォーマット形式	ext2 (購入したSSDによって違う)  
	オプション	rw,relatime (購入したSSDによって違う)  

3. パーティション/dev/sda1のUUIDを調べる

	pi@raspberrypi:~ $ `sudo blkid` ⏎   
	/dev/sda1: UUID="～" TYPE="ext2" PARTUUID=”～”  

4. 設定ファイル/etc/fstabの最下行に、上記を追記する

	UUID="～"　/mnt/ssd1　ext2　rw,relatime　0　2  

5. ラズベリー・パイを再起動する

	pi@raspberrypi:~ $ `sudo reboot` ⏎

### ReadyMediaをインストールする

1. メディア・サーバReadyMediaをインストールする

	pi@raspberrypi:~ $ `sudo apt-get install minidlna` ⏎

2. コンテンツ用ディレクトリを作成する

	pi@raspberrypi:~ $ `sudo mkdir -pm777 /mnt/ssd1/minidlna/pictures` ⏎
	pi@raspberrypi:~ $ `sudo mkdir -pm777 /mnt/ssd1/minidlna/music` ⏎
	pi@raspberrypi:~ $ `sudo mkdir -pm777 /mnt/ssd1/minidlna/videos` ⏎
	pi@raspberrypi:~ $ `sudo chown -R minidlna:minidlna /mnt/ssd1/minidlna/` ⏎

3. 設定ファイルにコンテンツのパスを記入する

	pi@raspberrypi:~ $ `sudo mousepad /etc/minidlna.conf` ⏎

	'''minidlna.conf
	media_dir=P,/mnt/ssd1/minidlna/pictures
	media_dir=A,/mnt/ssd1/minidlna/music
	media_dir=V,/mnt/ssd1/minidlna/videos
	'''

4. メディア・サーバReadyMediaを再起動する  

	pi@raspberrypi:~ $ `sudo service minidlna restart` ⏎

### パーティションを作成しなおす（上級者向け）  

1. マウントの解除  

	pi@raspberrypi:~ $ `sudo umount /dev/sda1` ⏎ 

2. fdiskの起動  

	pi@raspberrypi:~ $ `sudo fdisk /dev/sda` ⏎
	Welcome to fdisk (util-linux 2.33.1).
	～表示省略～

3. fdiskでパーティション削除  

	Command (m for help): `delete -a` ⏎
	Selected partition 1
	Partition 1 has been deleted.

3. fdiskでパーティション作成  

	Command (m for help): `new` ⏎
	Partition type
	   p   primary (0 primary, 0 extended, 4 free)
	   e   extended (container for logical partitions)
	Select (default p):⏎
	
	Using default response p.
	Partition number (1-4, default 1): ⏎
	First sector (2048-250069678, default 2048): ⏎
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-250069678, default 250069678): ⏎
	～表示省略～
	Do you want to remove the signature? [Y]es/[N]o: `Y` ⏎
	
	The signature will be removed by a write command.

3. fdiskでパーティション変更の実行  

	Command (m for help): `write` ⏎
	The partition table has been altered.
	Calling ioctl() to re-read partition table.
	Syncing disks.

4. mkfsでファイル・システムの作成  

	pi@raspberrypi:~ $ `sudo mkfs /dev/sda1` ⏎
	mke2fs 1.44.5 (15-Dec-2018)
	～表示省略～
	Allocating group tables: done
	Writing inode tables: done
	Writing superblocks and filesystem accounting information: done

5. 作成したボリュームの確認  

	pi@ raspberrypi:~ $ `ls /dev/sd*` ⏎
	/dev/sda  /dev/sda1


### 共有フォルダへの書き込みを許可する（上級者向け）

1. アカウントdmsuserを作成する  

	pi@raspberrypi:~ $ `sudo useradd dmsuser` ⏎  
	pi@raspberrypi:~ $ `sudo pdbedit -a dmsuser` ⏎  
	new password: ********** ⏎  
	retype new password: ********** ⏎  

2. 設定ファイルに[minidlna]を追加する  

	pi@raspberrypi:~ $ `sudo mousepad /etc/samba/smb.conf` ⏎
	
	```smb.conf
	[minidlna]
	path = /mnt/ssd1/minidlna
	browseable = yes
	read only = no
	guest ok = no
	```

## M5Cameraの準備

### Arduino IDEのセットアップ  
	https://www.arduino.cc/en/Main/Software  

- 本誌の実験用サンプル・プログラム：  
	https://github.com/bokunimowakaru/m5camera/archive/master.zip  

- ラズベリー・パイ用カメラ・サーバのダウンロードと実行  

	pi@raspberrypi:~ $ `git clone https://github.com/bokunimowakaru/m5camera` ⏎  
	pi@raspberrypi:~ $ `cd m5camera/tools` ⏎  
	pi@raspberrypi:~ $ `./get_photo_ssd.sh` ⏎  

## Nextcloud の準備

1. Nextcloudのインストール
	```
	sudo apt install apache2
	sudo apt install php
	sudo apt install mariadb-client
	sudo apt install mariadb-server
	cd /var/www/html
	sudo wget https://download.nextcloud.com/server/releases/nextcloud-19.0.3.zip
	sudo unzip nextcloud-19.0.3.zip
	cd nextcloud
	sudo mkdir -pm750 data
	sudo chown -R www-data:www-data data
	sudo chown -R www-data:www-data config
	sudo chown -R www-data:www-data apps
	```

2. PHPライブラリの追加（起動に必要）
	```
	sudo apt install php-mysql
	sudo apt install php-pgsql
	sudo apt install php-zip
	sudo apt install php-intl
	sudo apt install php-xml
	sudo apt install php-mbstring
	sudo apt install php-gd
	sudo apt install php-curl
	sudo service apache2 restart
	```
	上記はNextcloudの起動に必要な最小限度の構成です。他にも要件があるので、本格的に使用する際は下記の「Prerequisites for manual installation」をご覧ください。  
	https://docs.nextcloud.com/server/19/admin_manual/installation/source_installation.html

3. データベースへのアカウント追加（passwordは変更する）
	データベースと作成し、Nextcloudからのアクセス用のパスワードを設定します。
	```
	sudo mysql
	MariaDB [(none)]> create database nextcloud_db;
	MariaDB [(none)]> create user 'nxcuser'@'localhost' identified by 'password';
	MariaDB [(none)]> grant all privileges on nextcloud_db.* to 'nxcuser'@'localhost';
	MariaDB [(none)]> quit;
	```

4. SSD内にデータ用ディレクトリを作成
	```
	sudo mkdir -pm777 /mnt/ssd1/nextcloud
	sudo chown www-data:www-data /mnt/ssd1/nextcloud
	```

5. ブラウザでアクセスし、設定を行う。管理者アカウントはpiやnxcuserを避ける  
	http://127.0.0.1/nextcloud  
	```
	ユーザ名　：(他と違うユーザ名)
	パスワード：(他と違うパスワード)
	
	データフォルダー：/mnt/ssd1/nextcloud
	
	データベースのユーザ名　：nxcuser
	データベースのパスワード：password
	データベース名　　　　　：nextcloud_db
	```
しばらく「待機中」が続く、数分以上待つと、起動します（数分以上）。

- (参考) Nextcloud の初期化方法  
	★注意：すべての設定などが消えます。  
	実験で準備したNextcloudを作り直したいときは、以下のコマンドで、Nextcloudの環境を消去することが出来ます。
	```
	# Nextcloud の履歴や設定を初期化します(消えます)
	cd /var/www/html
	sudo rm -Rf nextcloud
	sudo unzip nextcloud-19.0.3.zip
	cd nextcloud
	sudo mkdir -pm750 data
	sudo chown -R www-data:www-data data
	sudo chown -R www-data:www-data config
	sudo chown -R www-data:www-data apps
	sudo mysql
	MariaDB [(none)]> drop database nextcloud_db;
	MariaDB [(none)]> create database nextcloud_db;
	MariaDB [(none)]> quit;
	sudo service apache2 restart
	```

- (参考) データベースのroot用のパスワードを設定するには以下のコマンドを入力します。  
	```
	MariaDB [(none)]> update mysql.user set password=password('password1') where user = 'root';
	MariaDB [(none)]> flush privileges;
	```
- (参考) セキュリティ自己診断機能  
　ユーザアイコンの［設定］をクリックし、左側のサイド・メニュー（表示されない場合は、ウィンドウ幅を広げるかメニューアイコンをクリック）の項目「管理」の［概要］をクリックすると自己診断が実行されます。  
　サーバをインターネットに公開する場合は、必ず診断結果を確認して下さい。  
　著作権のあるコンテンツをインターネットに公開すると、仮に誤操作や不具合であったとしても、権利者への賠償責任と法律による刑事罰の対象となる場合があります。  

## サーバとして使用する際の注意点

ラズベリー・パイは、プログラミング学習用として設計されました。
ホーム・サーバとして常時通電した動作は、想定されていません。
また、M5CameraやT-Cameraについても安全性や信頼性が確保されていない実験用の製品です。  
これらの製品を常時・通電して運用する場合は、自己責任で安全性や信頼性、合法性などに留意する必要があります．
具体的には、ラズベリー・パイやM5Camera、T-Camera、周辺機器、ACアダプタなどによる火災を起こさないための対策や、
インターネットからの脅威に対する備え、著作権のあるコンテンツの適切な管理、マイクロSDカードやSSDなどの寿命に対する考慮が必要です。

1. 火災を起こさないための対策
2. インターネット・セキュリティ対策
3. 著作権に対する管理
4. データ破損に対する対策

とくに、上記1～3については、自分だけでなく他人や社会を巻き込む恐れがあるので、慎重に取り扱ってください。

### セキュリティ対策の一例

1. 機器ごと、用途ごとにアカウントとパスワードを設定（同じパスワードを共用しない）
2. ファイヤーウォールの設置（ホーム・ゲートウェイ、ラズベリー・パイ本体など）
3. ファイルの読み書き実行権限の設定
4. OS、アプリケーション、ライブラリの定期的な更新
5. 不正侵入や不正操作の監視（ログイン履歴、アクセス履歴の保存と拒絶時のアラート送信など）

by 国野 亘 <https://bokunimo.net>

[https://git.bokunimo.com/nas/](https://git.bokunimo.com/nas/)
