# YOLOv5-Realtime-Cam-Detection-on-MIIVII_Lite_NX_II-NVIDIA-Jetson-Xavier-NX



### 挂载磁盘（U盘、移动硬盘）

```
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



```
nvidia@miivii-tegra:~$ sudo fdisk -l | grep "G"
Disk /dev/mmcblk0: 14.7 GiB, 15758000128 bytes, 30777344 sectors
/dev/mmcblk0p1        40 29360167 29360128    14G Microsoft basic data

nvidia@miivii-tegra:~$ sudo fdisk -l | grep "G"
Disk /dev/mmcblk0: 14.7 GiB, 15758000128 bytes, 30777344 sectors
/dev/mmcblk0p1        40 29360167 29360128    14G Microsoft basic data
Disk /dev/mmcblk1: 119.1 GiB, 127865454592 bytes, 249737216 sectors
/dev/mmcblk1p1      32768 249737215 249704448 119.1G  7 HPFS/NTFS/exFAT
```

```
test:
/dev/mmcblk1p1 /mnt/workspace xfs defaults,usrquota,grpquota 0 0 
```



```
sudo apt-get install exfat-fuse

sudo mkdir /media/nvidia/workspace
sudo mount /dev/mmcblk1p1 /media/nvidia/workspace

nvidia@miivii-tegra:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/mmcblk0p1   14G   12G  1.1G  92% /
none            3.5G     0  3.5G   0% /dev
tmpfs           3.8G   40K  3.8G   1% /dev/shm
tmpfs           3.8G   38M  3.8G   1% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           3.8G     0  3.8G   0% /sys/fs/cgroup
tmpfs           778M   12K  778M   1% /run/user/120
tmpfs           778M  132K  778M   1% /run/user/1000
/dev/mmcblk1p1  120G   17M  120G   1% /media/nvidia/workspace
```



## 刷sd

```
df -Th
sudo fdisk -l | grep "G"
umount /dev/mmcblk1p1
sudo mkfs.ext4 /dev/mmcblk1p1
sudo shutdown -r now

df -Th
/media/nvidia/2db574f1-f146-4ea3-ab2c-8b2a7b6a5e9d
```



### docker



```
nvidia@miivii-tegra:~$ docker --version
Docker version 19.03.6, build 369ce74a3c

nvidia@miivii-tegra:~/Desktop$ sudo docker info | grep "Docker Root Dir"
Docker Root Dir: /var/lib/docker
```

```

sudo docker ps | awk '{print $1}' |xargs docker stop
sudo systemctl stop docker

sudo cp -r /var/lib/docker /var/lib/docker.bak
sudo mkdir -p /media/nvidia/2db574f1-f146-4ea3-ab2c-8b2a7b6a5e9d/workspace/data
sudo mv /var/lib/docker /media/nvidia/2db574f1-f146-4ea3-ab2c-8b2a7b6a5e9d/workspace/data/
sudo ln -s /media/nvidia/2db574f1-f146-4ea3-ab2c-8b2a7b6a5e9d/workspace/data/docker /var/lib
sudo chmod 777 -R /media/nvidia/2db574f1-f146-4ea3-ab2c-8b2a7b6a5e9d/workspace/data/docker
sudo systemctl start docker
```

```
old:
sudo docker ps | awk '{print $1}' |xargs docker stop
sudo systemctl stop docker

# sudo cp -r /var/lib/docker /var/lib/docker.bak
sudo mkdir -p /mnt/workspace/data
sudo mv /var/lib/docker /mnt/workspace/data/
sudo ln -s /mnt/workspace/data/docker /var/lib
sudo chmod 777 -R /mnt/workspace/data/docker
sudo systemctl start docker
```





Local ?

```
sudo mv /usr/local /usr/local.bak
sudo mkdir -p /mnt/workspace/usr/local
sudo cp -r /usr/local.bak/* /mnt/workspace/usr/local
sudo ln -s /mnt/workspace/usr/local /usr/

```





查看jetpack版本

```
cat /etc/nv_tegra_release 
```

out

```
# R32 (release), REVISION: 5.0, GCID: 25531747, BOARD: t186ref, EABI: aarch64, DATE: Fri Jan 15 23:21:05 UTC 2021
```



```
https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-pytorch
```

JetPack 4.5 (L4T R32.5.0)

```
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



```
sudo docker pull nvcr.io/nvidia/l4t-pytorch:r32.5.0-pth1.7-py3


```



### 



```
sudo mkdir -p /mnt/workspace/data/docker_container/pytorch
sudo chmod 777 -R /mnt/workspace/data/docker_container/pytorch
cd /mnt/workspace/data/docker_container/pytorch

sudo git clone https://github.com/LanXiu0523/YOLOv5-Realtime-Cam-Detection-on-MIIVII_Lite_NX_II-NVIDIA-Jetson-Xavier-NX.git

sudo docker run -it --rm --runtime nvidia --network host -v /mnt/workspace/data/docker_container/pytorch:/location/in/container nvcr.io/nvidia/l4t-pytorch:r32.5.0-pth1.7-py3

cd /location/in/container/YOLOv5-Realtime-Cam-Detection-on-MIIVII_Lite_NX_II-NVIDIA-Jetson-Xavier-NX/yolov5/


apt-get update -y
apt-get dist-upgrade -y
apt-get install build-essential cmake -y

python3 -m pip install -U pip -i https://pypi.tuna.tsinghua.edu.cn/simple 
python3 -m pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
python3 -m pip install pyproject==1.3.1
python3 -m pip install -r requirements.txt --verbose

python3 detect.py --source 0  


```






