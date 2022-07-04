# 本ページでは
このページではraspicat2で作業するときに必要となる情報をまとめます。(slackだと一覧性が著しく悪いため)

# RasPiCatの基本情報

|  |初代RasPiCat|2代目RasPiCat|
|---| ------------- | ------------- |
|RaspberryPi|4B 4GB|4B 4GB |
|Lider|VLP-16|UTM-30LX|
|認識距離|100m|30m(最大は60m)|
|視野角|水平360° 垂直30°|270°|
|スキャン時間|50~200ms|25ms|
|測定精度|±３cm（1σ@25m)|0.1～10m：±30mm 10～30m：±50mm|
|製品HP|https://www.argocorp.com/cam/special/Velodyne/VLP-16.html|https://www.hokuyo-aut.co.jp/search/single.php?serial=21|

# SSDのマウント設定

`cd /mnt/ssd`

で***ルートディレクトリ直下***のmntファイルにアクセスできる

ssdの中身が見れてるか確認

   ~~Raspberry Pi 4B
1.    `sudo fdisk -l`(SSDの場所が/dev/sda1と分かる)
2. `sudo mkdir /mnt/SSD`
3. `sudo mount -o uid=1000,gid=1000 /dev/sda1 /mnt/SSD`~~
4. `df -T`でマウント確認
5. 外すときは
   
   `sudo umonunt /mnt/SSD`

# rosbagを用いたmapping
1.Raspberry Pi 4B 

    roslaunch raspicat_navigation raspicat_bringup.launch urg_ether:=false urg_usb:=true waypoint_navigation:=false urg_serial_port:=/dev/ttyACM0 

2.Joystick controllerを接続している方   

    roslaunch raspicat_gamepad_controller logicool.launch

3.***Rspberry Pi側の /mnt/SSD***   

    rosbag record -O 建物の名前.bag /cmd_vel /odom /scan /tf /tf_static

   `rosbag record -j -O 建物の名前.bag /cmd_vel /odom /scan /tf /tf_static`(圧縮)

   
4.`sudo umount /mnt/SSD`

### ノートpc側で確認したいとき
`roslaunch raspicat_slam_navigation slam_remote_pc.launch`

# 更新後rosbag取得
Raspberry Pi 4B (launch立ち上げ)
1. `roslaunch raspicat_bringup raspicat_bringup.launch urg:=true joy:=true`
2. `rostopic list`

無線操作するならPC上で
1. `roslaunch raspicat_gamepad_controller logicool.launch`

Raspberry Pi 4B　(/mnt/ssd/test.bag取得)
1. `cd /mnt/ssd`
2. `rosbag record -O test.bag /cmd_vel /odom /scan /tf /tf_static`
    (全データ取得するなら`rosbag record -O test.bag -a`)
3. `rosbag info test.bag`

PCから無線ファイル編集
1. `sftp ubuntu@192.168.12.1`
2. `get /mnt/ssd/test.bag`
3. `bye`

# 更新後マップ作成
Gmapping
1. `roslaunch raspicat_slam raspicat_gmapping.launch rosbag:=true rosbag_rate:=1.0 rosbag_filename:=$HOME/map/test.bag`
   `rosbag_rate:=`でデータを回す速度調整


# .bagをRvizで再生する
1.  `rosbag　decompress　~.bag`(解凍)
2.  .bashrcのipをlocalhostに直す
3.  `roscore`
4.  `rosparam set /use_sim_time true`
5.  `roslaunch raspicat_slam_navigation slam_remote_pc.launch`
6.  rosbagファイルがあるところ   
    `rosbag play ~.bag -r 1 --clock`   
1. `rosrun map_server map_saver -f ~/map`

以下のファイルでパラメータ設定   
/raspicat_ws/src/raspicat_slam_navigation/launch/slam_remote_pc.launch'

# .bagファイルの圧縮と解凍
* `rosbag record -j`でbagファイルに保存しながら圧縮   
`rosbag record -j /cmd_vel /odom /scan /tf /tf_static`   
* 指定したbagファイルを圧縮   
`rosbag compress ~.bag`   
(圧縮前のファイルも.orig.bagとしてバックアップされるらしい)   
* 指定したbagファイルを解凍(推奨)   
`rosbag decompress ~.bag`   

# .bagファイルが取り出せなくなった方向け

「fu○k SSDをマウントしたつもりがされていなかった」なんて人向けです

1. ファイルアプリを開く
2. 左のメニューからその他の場所を選択
3. 下にある他のサーバーに接続に``sftp://192.168.12.1/``と打ち、接続
4. みごとできましたぁ

参考:https://www.indoorcorgielec.com/resources/raspberry-pi/raspberry-pi-ssh/

# VLP-16の自動接続設定
VLP-16は初期設定を完了しても、コンピュータを再起動すると再度コマンドで接続させる必要があります。この手間を解消します。  
ここで紹介する方法では、NetworkManagerコマンドラインインターフェースを用います。  
まず、VLP-16用の接続プロファイルを新規作成します。  
```
sudo nmcli connection add type ethernet ifname eth0 con-name vlp-16 autoconnect yes ipv6.method ignore
```
ipv4アドレスを手動に設定します。  
```
sudo nmcli connection modify vlp-16 ipv4.method manual
```
VLP-16にipアドレスを割り当てます。  
```
nmcli connection modify vlp-16 ipv4.address "192.168.0.100/24"
```
VLP-16のプロファイルに静的ルートを設定します。
```
nmcli connection modify vlp-16 ipv4.routes "192.168.0.201"
```

これで完了です。再起動で自動的に接続されます。  
VLP-16の初期設定は以下を参考に完了しているものとします。  
http://wiki.ros.org/velodyne/Tutorials/Getting%20Started%20with%20the%20Velodyne%20VLP16





