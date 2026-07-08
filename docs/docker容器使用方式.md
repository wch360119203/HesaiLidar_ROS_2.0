# docker容器使用方式

本文档说明如何在 Docker 容器中编译、运行和调试 Hesai ROS2 驱动。当前配置使用 ROS2 Humble，宿主机系统为 Windows。

## 目录挂载

容器会挂载以下宿主机目录：

```text
C:\code\HesaiLidar_ROS_2.0 -> /workspaces/hesai_ws/src/hesai_ros_driver
C:\code\HesaiLidar_SDK_2.0 -> /workspaces/hesai_ws/src/hesai_ros_driver/src/driver/HesaiLidar_SDK_2.0
C:\code\data -> /data
```

其中：

- ROS 驱动源码在 `/workspaces/hesai_ws/src/hesai_ros_driver`
- SDK 源码在 `/workspaces/hesai_ws/src/hesai_ros_driver/src/driver/HesaiLidar_SDK_2.0`
- pcap 和标定文件在 `/data`

当前已验证的数据文件路径：

```text
/data/input.pcap
/data/AT128P_AngleCalibration.dat
```

## 启动容器

如果之前进入过旧容器，先清理旧容器，避免继续使用缺少 SDK 挂载或 ROS 环境变量的实例：

```bash
docker compose down --remove-orphans
```

构建镜像：

```bash
docker compose build
```

进入开发容器：

```bash
docker compose run --rm hesai-dev
```

进入容器后，可以先确认 ROS2 环境和 SDK 挂载是否正常：

```bash
echo $ROS_VERSION
echo $ROS_DISTRO
ls /workspaces/hesai_ws/src/hesai_ros_driver/src/driver/HesaiLidar_SDK_2.0/CMakeLists.txt
```

正常情况下应输出：

```text
2
humble
/workspaces/hesai_ws/src/hesai_ros_driver/src/driver/HesaiLidar_SDK_2.0/CMakeLists.txt
```

## 编译

在容器内执行：

```bash
cd /workspaces/hesai_ws
rosdep update
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

如果编译成功，会看到类似输出：

```text
Finished <<< hesai_ros_driver
Summary: 1 package finished
```

## 配置 pcap 回放

调试 pcap 前，修改 `config/config.yaml`。

将数据源切换为 pcap：

```yaml
source_type: 2
```

将 `pcap_type` 配置为容器内路径：

```yaml
pcap_type:
  pcap_path: "/data/input.pcap"
  correction_file_path: "/data/AT128P_AngleCalibration.dat"
  firetimes_path: ""
  pcap_play_synchronization: true
  pcap_play_in_loop: false
  play_rate_: 1.0
```

## 运行节点

编译并 source 后，可以直接启动节点：

```bash
source /workspaces/hesai_ws/install/setup.bash
ros2 run hesai_ros_driver hesai_ros_driver_node
```

也可以使用项目自带 launch 文件：

```bash
source /workspaces/hesai_ws/install/setup.bash
ros2 launch hesai_ros_driver start.py
```

注意：`start.py` 会同时启动 `rviz2`。如果 Windows Docker 没有配置图形界面转发，RViz 可能报显示相关错误；只验证 pcap 解析和点云发布时，优先使用 `ros2 run hesai_ros_driver hesai_ros_driver_node`。

## 验证 topic

建议使用一个长期运行的容器，方便另开终端查看 ROS topic。

第一个终端启动容器：

```bash
docker compose up -d hesai-dev
docker compose exec hesai-dev bash
```

在第一个容器终端中运行节点：

```bash
cd /workspaces/hesai_ws
source install/setup.bash
ros2 run hesai_ros_driver hesai_ros_driver_node
```

第二个终端进入同一个容器：

```bash
docker compose exec hesai-dev bash
```

查看 topic：

```bash
source /workspaces/hesai_ws/install/setup.bash
ros2 topic list
ros2 topic echo /lidar_points --once
```

常用 topic 包括：

```text
/lidar_points
/lidar_imu
/lidar_packets_loss
```

停止容器：

```bash
docker compose down
```

## 常见问题

### ROS_VERSION 为空

现象：

```text
-- ROS_VERSION is
```

处理方式：退出旧容器，执行：

```bash
docker compose down --remove-orphans
docker compose run --rm hesai-dev
```

### SDK 目录没有 CMakeLists.txt

现象：

```text
HesaiLidar_SDK_2.0 does not contain a CMakeLists.txt file
```

处理方式：确认宿主机目录 `C:\code\HesaiLidar_SDK_2.0` 存在，并且容器内可以看到：

```bash
ls /workspaces/hesai_ws/src/hesai_ros_driver/src/driver/HesaiLidar_SDK_2.0/CMakeLists.txt
```

如果看不到，清理旧容器后重新进入：

```bash
docker compose down --remove-orphans
docker compose run --rm hesai-dev
```
