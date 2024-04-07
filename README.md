# YOLOv5-Realtime-Cam-Detection-on-MIIVII_Lite_NX_II-NVIDIA-Jetson-Xavier-NX

## 挂载磁盘

```bash
nvidia@miivii-tegra:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/mmcblk0p1   14G   12G  1.2G  92% /
none            3.5G     0  3.5G   0% /dev
tmpfs           3.8G   40K  3.8G   1% /dev/shm
tmpfs           3.8G   38M  3.8G   1% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           3.8G     0  3.8G   0% /sys/fs/cgroup
tmpfs           778M   12K  778M   1% /run/user/120
tmpfs           778M  132K  778M   1% /run/user/1000
```

```bash
# 插入磁盘前
nvidia@miivii-tegra:~$ sudo fdisk -l | grep "G"
Disk /dev/mmcblk0: 14.7 GiB, 15758000128 bytes, 30777344 sectors
/dev/mmcblk0p1        40 29360167 29360128    14G Microsoft basic data
# 插入磁盘后
nvidia@miivii-tegra:~$ sudo fdisk -l | grep "G"
Disk /dev/mmcblk0: 14.7 GiB, 15758000128 bytes, 30777344 sectors
/dev/mmcblk0p1        40 29360167 29360128    14G Microsoft basic data
Disk /dev/mmcblk1: 119.1 GiB, 127865454592 bytes, 249737216 sectors
/dev/mmcblk1p1      32768 249737215 249704448 119.1G  7 HPFS/NTFS/exFAT
```

### 格式化磁盘

```bash
df -Th

sudo fdisk -l | grep "G"
umount /dev/mmcblk1p1
sudo mkfs.ext4 /dev/mmcblk1p1

sudo mkdir /mnt/workspace
sudo mount /dev/mmcblk1p1 /mnt/workspace

sudo blkid
# 记录磁盘UUID /dev/mmcblk1p1: UUID="d5e6af59-c988-4f44-9acd-6f1a23f972ad" TYPE="ext4"

sudo vim /etc/fstab
# 末尾添加
UUID=d5e6af59-c988-4f44-9acd-6f1a23f972ad  /mnt/workspace  ext4  defaults  0  2 

sudo shutdown -r now

df -Th
```



### 配置Docker存储路径到挂载磁盘

```bash
# 查看Docker版本
nvidia@miivii-tegra:~$ docker --version
Docker version 19.03.6, build 369ce74a3c
# 查看Docker当前存储目录
nvidia@miivii-tegra:~/Desktop$ sudo docker info | grep "Docker Root Dir"
Docker Root Dir: /var/lib/docker

sudo docker ps | awk '{print $1}' |xargs docker stop
sudo systemctl stop docker
sudo cp -r /var/lib/docker /var/lib/docker.bak

sudo mkdir -p /mnt/workspace/data
sudo mv /var/lib/docker /mnt/workspace/data/
sudo ln -s /mnt/workspace/data/docker /var/lib
sudo chmod 777 -R /mnt/workspace/data/docker
sudo systemctl start docker

```



## 拉取YOLOv5的Docker镜像

```bash
sudo mkdir -p /mnt/workspace/data/docker_container/pytorch
cd /mnt/workspace/data/docker_container/pytorch

sudo docker pull lanxiu0523/yolov5-realtime-cam-detection-on-miivii_lite_nx2-nvidia-jetson-xavier-nx:r32.5.0-py3.6-pytorch1.7-opencv4.5.5.62_v1.0

# 启动docker并进入
# 需要连接摄像头（没有摄像头则去掉‘--device=/dev/video0 \’一行）
sudo docker run -it \
--runtime nvidia \
--network host \
--name pytorch \
--device=/dev/video0 \
-v /mnt/workspace/data/docker_container/pytorch:/location/in/container \
lanxiu0523/yolov5-realtime-cam-detection-on-miivii_lite_nx2-nvidia-jetson-xavier-nx:r32.5.0-py3.6-pytorch1.7-opencv4.5.5.62_v1.0

# 按住Ctrl，先按p后按q，脱离docker
```



## 运行YOLOv5的Detect

```bash
# 在docker外
sudo git clone https://github.com/LanXiu0523/YOLOv5-Realtime-Cam-Detection-on-MIIVII_Lite_NX_II-NVIDIA-Jetson-Xavier-NX.git

# 进入docker
sudo docker attact pytorch

cd /location/in/container/YOLOv5-Realtime-Cam-Detection-on-MIIVII_Lite_NX_II-NVIDIA-Jetson-Xavier-NX/yolov5/

apt-get update -y
apt-get dist-upgrade -y
apt-get install build-essential cmake -y

python3 -m pip install -U pip -i https://pypi.tuna.tsinghua.edu.cn/simple 
python3 -m pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
python3 -m pip install pyproject==1.3.1
python3 -m pip install -r requirements.txt --verbose

# 图像测试（无需摄像头）
python3 detect.py

# 相机实时目标检测（需连接摄像头）
python3 detect.py --source 0  
```



## 附录

### 为MIIVII安装基于Jetpack的NVIDIA官方镜像

**查看L4T版本**

```bash
cat /etc/nv_tegra_release 
```

**根据输出，L4T版本为 `R32.5.0`**

```bash
# R32 (release), REVISION: 5.0, GCID: 25531747, BOARD: t186ref, EABI: aarch64, DATE: Fri Jan 15 23:21:05 UTC 2021
```

**阅读NVIDIA官方文档，得知 `L4T R32.5.0` 对应Jetpack版本为 `JetPack 4.5`**

```bash
https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-pytorch
```

**并得知NVIDIA官方提供的 `JetPack 4.5 (L4T R32.5.0)` 的PyTorch镜像可用版本如下**

```bash
JetPack 4.5 (L4T R32.5.0)
	l4t-pytorch:r32.5.0-pth1.6-py3
		PyTorch v1.6.0
		torchvision v0.7.0
		torchaudio v0.6.0
	l4t-pytorch:r32.5.0-pth1.7-py3
		PyTorch v1.7.0
		torchvision v0.8.0
		torchaudio v0.7.0
```

**拉取镜像**

```bash
sudo docker pull nvcr.io/nvidia/l4t-pytorch:r32.5.0-pth1.6-py3
# or
sudo docker pull nvcr.io/nvidia/l4t-pytorch:r32.5.0-pth1.7-py3
```

**启动并进入镜像**

```bash
sudo docker run -it --rm --runtime nvidia --network host nvcr.io/nvidia/l4t-pytorch:r32.5.0-pth1.6-py3
#or
sudo docker run -it --rm --runtime nvidia --network host nvcr.io/nvidia/l4t-pytorch:r32.5.0-pth1.7-py3
```

