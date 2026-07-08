# docker容器使用方式

使用方式：

``` bash
docker compose build
docker compose run --rm hesai-dev
```

如果之前已经进入过旧容器，先退出并清理旧容器，避免继续使用缺少 SDK 挂载的实例：

``` bash
docker compose down --remove-orphans
```

容器会挂载以下宿主机目录：

``` bash
C:\code\HesaiLidar_ROS_2.0 -> /workspaces/hesai_ws/src/hesai_ros_driver
C:\code\HesaiLidar_SDK_2.0 -> /workspaces/hesai_ws/src/hesai_ros_driver/src/driver/HesaiLidar_SDK_2.0
C:\code\data -> /data
```

进入容器后

``` bash
rosdep update
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

你的 pcap 数据在容器里对应：

``` bash
/data/input.pcap
/data/AT128P_AngleCalibration.dat
```

调试 pcap 前，需要把 config.yaml 里的 source_type 改成 2，并把 pcap_path、correction_file_path 指向 /data/...。
