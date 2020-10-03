# nas : ラズベリー・パイで作るホーム・メディア・サーバ by 国野 亘

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

	```/etc/samba/smb.conf
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
	
	media_dir=P,/mnt/ssd1/minidlna/pictures
	media_dir=A,/mnt/ssd1/minidlna/music
	media_dir=V,/mnt/ssd1/minidlna/videos

4. メディア・サーバReadyMediaを再起動する

	pi@raspberrypi:~ $ `sudo service minidlna restart` ⏎

### パーティションを作成しなおす（上級者向け）

	pi@raspberrypi:~ $ `sudo umount /dev/sda1` ⏎ 
	pi@raspberrypi:~ $ `sudo fdisk /dev/sda` ⏎
	Welcome to fdisk (util-linux 2.33.1).
	～表示省略～
	
	Command (m for help): `delete -a` ⏎
	Selected partition 1
	Partition 1 has been deleted.
	
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
	
	Command (m for help): `write` ⏎
	The partition table has been altered.
	Calling ioctl() to re-read partition table.
	Syncing disks.
	
	pi@raspberrypi:~ $ `sudo mkfs /dev/sda1` ⏎
	mke2fs 1.44.5 (15-Dec-2018)
	～表示省略～
	Allocating group tables: done
	Writing inode tables: done
	Writing superblocks and filesystem accounting information: done
	
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
	
	```/etc/samba/smb.conf
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
	mkdir data
	sudo chown www-data:www-data /var/www/html/nextcloud/data
	sudo chown www-data:www-data config
	sudo chown www-data:www-data apps
	```

2. PHPライブラリの追加
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

3. データベースへのアカウント追加
	```
	sudo mysql
	MariaDB [(none)]> create database nextcloud_db;
	MariaDB [(none)]> use nextcloud_db;
	MariaDB [nextcloud_db]> create user 'nxcuser'@'localhost' identified by 'password';
	MariaDB [nextcloud_db]> grant all privileges on nextcloud_db.* to 'nxcuser'@'localhost';
	```

by <https://bokunimo.net>
