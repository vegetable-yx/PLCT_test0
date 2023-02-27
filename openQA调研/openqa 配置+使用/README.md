# openQA部署及简单使用

### 环境（仅供参考）

openSUSE Tumbleweed x86_64

内存大小：4GB

存储大小：50GB

*如不太熟悉虚拟机安装过程可以参考如下教程：*[opensuse虚拟机安装教程](https://blog.csdn.net/weixin_43020570/article/details/117064994)



## 部署openQA

1. 安装 openQA-bootstrap 包

```
zypper in openQA-bootstrap
cd /usr/share/openqa/script/
./openqa-bootstrap
```

经过本人测试，第一次执行改脚本基本上都会异常退出，可以修改一下 /usr/share/openqa/script/fetchneedles 文件：

5-7行替换为：

```bash
: "${dist_name:=${dist:-"openEuler"}}" # the display name, for the help message
: "${dist:="openeuler"}"
: "${giturl:="https://gitee.com/lvxiaoqian/os-autoinst-distri-openeuler.git"}"
```

15行替换为：

```bash
: "${needles_giturl:="https://gitee.com/lvxiaoqian/os-autoinst-needles-openeuler.git"}"
```

![]()

然后再次执行改脚本即可。

2. 修改 qemu.pm

此外，`/usr/lib/os-autoinst/backend/qemu.pm` 文件需要被修改，打上这个patch `https://gitee.com/lvxiaoqian/os-autoinst/commit/c2c580265330c7bab82e2efe61ebd8612671a0eb.patch` 或者直接将改文件内容替换为以下内容：[qemu.pm]()

这时 openQA web UI 已经可以在 http://localhost/ 打开，首先可以以demo的身份登录（不需要输入密码）。此时的 openqa-bootstrap 已经启动了一个本地的worker（openqa-bootstrap 自动开启的），在workers界面可以看到它。

![]()

![]()

启动本地riscv worker
---

安装 `qemu-system-riscv64` 包

```
zypper in qemu-extra
```

修改/etc/openqa/workers.ini，加入以下内容，其中10是worker的编号，可以修改

```
[10]
WORKER_CLASS=qemu_riscv64
QEMU_NO_KVM=1
QEMUCPU=rv64
QEMUMACHINE=virt,usb=off
```

启动worker

```
systemctl enable --now openqa-worker@10
```

在 worker 界面刷新一下，即可看到新加的 10 号 

![]()

### 配置Medium types

点击右上角`logged in as Demo`会出现下拉菜单，可以进入各个菜单配置，点击Medium types即可进行Medium types的配置。

点击 new Medium Types，然后输入以下信息：

![]()

**具体配置信息：**

Distri: openeuler

Version: 22.03

Flavor: v2

arch: riscv64

![]()

### 配置Machines

点击 new Machines，然后输入以下信息：

**具体配置信息：**

Name: RISCV64_VM

Backend: qemu

Settings：

```
APPEND=root=/dev/vda1
BIOS=none
CDMODEL=virtio-blk-device
HDDMODEL=virtio-blk-device
HDD_1=openEuler-22.03-V2-xfce-qemu-testing.qcow2
KERNEL=fw_payload_oe_qemuvirt.elf
NOAUTOLOGIN=1
NUMDISKS=1
PASSWORD=openEuler12#$
QEMU=riscv64
QEMUCPU=rv64
QEMUCPUS=4
QEMUMACHINE=virt,accel=kvm,usb=off,dump-guest-core=off,gic-version=3
QEMURAM=4096
ROOTONLY=1
SERIALDEV=ttyS0
TIMEOUT_SCALE=2
VGA=std
VIRTIO_CONSOLE=1
WORKER_CLASS=qemu_riscv64
```

![]() 

### 配置Test suites

点击 new Test suites，然后输入以下信息：

**具体配置信息：**

Name: install_testsuite

Setting:

```
DESKTOP=xfce
HDDSIZEGB=100
INSTALL=1
NICTYPE=user
YAML_SCHEDULE=schedule/install_testsuite.yaml
```

![]() 

### 配置Job groups

点击“+”添加新的`job group`到当前层级

![]()

然后点击进入新家的 job，编辑 job template

```
---

defaults:
  riscv64:
    machine: RISCV64_VM
    priority: 50

products:
  openeuler-v2-riscv64-22.03:
     distri: openeuler
     flavor: v2
     version: 22.03
scenarios:
  riscv64:
     openeuler-v2-riscv64-22.03:
        - install_testsuite
```

- `machine`为之前在`Machines`中定义的`RISCV64_VM`，指明在哪个machine上面跑
- `products`为之前在`Medium`中定义的`openeuler`，`openeuler-v2-riscv64-22.03`相当于起了一个名字，后续在scenario中使用
- `scenarios`中配置在`openeuler-v2-riscv64-22.03`系统的`RISCV64_VM`虚拟机中跑的测试集
- `install_testsuite`为之前在`Test suite`中定义的`Name`，指明测试套

![]()

执行测试
---

安装qemu-system-riscv64包：

```
zypper install qemu-extra
```

在对应路径下下载镜像

```
cd /var/lib/openqa/share/factory/hdd
wget https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/testing/20221212/v0.1/QEMU/openEuler-22.03-V2-xfce-qemu-testing.qcow2.tar.zst
tar -I 'zstdmt' -xvf openEuler-22.03-V2-xfce-qemu-testing.qcow2.tar.zst
cd /var/lib/openqa/share/factory/other
wget https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/testing/20221212/v0.1/QEMU/fw_payload_oe_qemuvirt.elf
```

执行测试

```
openqa-cli api -X POST isos async=0 DISTRI=openeuler FLAVOR=v2 ARCH=riscv64 VERSION=22.03
```

点击左上角Job Groups，可以看到build结果


![]()

![]()

点击build进入结果页面，这里之前`openqa-cli api -X POST isos`的结果都会列出
![]()

点击结果按钮进入详情页面
![]()此页面可以看到该用例的测试步骤，以及详细的检查点

- 点击`Logs&Asserts`可以看到详细的日志
- 点击`Settings`可以看到用例设置的变量

![]()


