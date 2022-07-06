# 起動コマンドでござる

## rosbag
### rosbag record
1.マウントされている***ルートディレクトリ直下***のmntファイルにアクセスする

`cd /mnt/ssd`

2.RaspberryPiCatと通信ができているか確認

できていなかったら→~/.bashrc書き換え

`rostopic list`

3.記録開始

`rosbag record -a -O .bag`

#### option
/topic:topicを指定して記録

-a:すべてのtopicを記録

-o:名称 日付.bagとなる

-O:名称.bagとなる

-e:正規表現を用いてtopicを抽出

-d:bagファイルの最大時間を設定

-split:bagファイルの最大サイズを設定

-j:BZ2圧縮

-lz4:LZ4圧縮

### 無線でRaspberryPiCatからファイルを取り出す方法

1. ファイルアプリを開く
2. 左のメニューバーの一番下からその他の場所を選択
3. 右下にある他のサーバーに接続に``sftp://192.168.12.1/``と打ち、接続


### rosbag info
-y /topic/.bag:yaml形式でprint


### rosbag play
0.bagファイルを解凍する
`rosbag decompress .bag`
1.`rosbag play .bag -r 1 --clock`

#### option

-q:コンソール出力を制限

--pause:一時停止モードで開始

--clock:時刻を公開

-s:〜秒後から再生

-u:〜秒だけ再生

-l:ループ再生

--topics /topic/:再生するtopicを指定




## コントローラで動かすだけ
`roslaunch raspicat_bringup raspicat_bringup.launch joy:=true`


## マッピング
### マッピング起動
### rosbagでマップ生成


## ナビゲーション
### way-point設定
`roslaunch raspicat_waypoint_navigation waypoint_set.launch map_file:="$HOME/編集後マップの場所.yaml"`

raspicat_ws/src/raspicat/raspicat_navigation/raspicat_waypoint_navigation/config/waypoint/waypoin.yaml
が更新される

### ナビゲーション起動
1.`roslaunch raspicat_waypoint_navigation raspicat_raspi_only_navigation.launch urg:=true mcl_init_pose_x:=116.629730 mcl_init_pose_y:=10.296697 mcl_init_pose_a:=3.10499862591029 initial_pose_x:=116.629730 initial_pose_y:=10.296697 initial_pose_a:=3.10499862591029 mcl_map_file:=$HOME/waypoint/1/1.yaml move_base_map_file:=$HOME/waypoint/1/2.yaml waypoint_yaml_file:=$HOME/waypoint/1/3.yaml`

座標は初期位置（Rvizで確認）
mcl_map_fileは自己位置推定に使うマップ
move_base_mapは編集後のマップ
waypoint_yaml_fileはway-point設定を行ったマップ

2.pcで確認
~/raspicat_ws/src/raspicat/raspicat_navigation/raspicat_waypoint_navigation/config/rviz

`rosrun rviz rviz  -d raspicat_waypoint_navigation.rviz`


3.ナビゲーション開始

`rostopic pub -1 /way_nav_start std_msgs/Empty`

