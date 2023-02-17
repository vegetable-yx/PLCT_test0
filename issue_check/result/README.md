# RISC-V oE preview V2镜像 Retest mugen自动化测试结果说明

## 测试说明

### 测试范围

- 共323个测试套(321个软件包+systemd+os-basic)，1426个测试用例  
  
  ### 测试框架和测试方法
- 测试框架：mugen-riscv   
- 测试方式：测试环境自动复原，测试套间隔离以及自动分配硬盘外设资源的自动化测试  
    使用qemu_test.py，利用qemu qcow2快照实现在测试每个测试套时单独建立qemu虚拟机进行测试，保证了测试套间不会相互干扰。测试程序运行时会根据测试套文件中"add disk"字段的信息，自动创建硬盘资源并分配给对应的虚拟机。  
- 测试用例代码位置：https://github.com/brsf11/mugen-riscv/tree/riscv/testcases/cli-test/对应测试套名/对应测试用例名.sh  

### 测试环境

- 本次测试使用镜像：https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/preview/openEuler-22.03-V2-riscv64/QEMU/  
- 软件源：默认  
- CPU核数：8  
- 内存大小：8G  
- qemu启动参数：  
  
  ```shell
  qemu-system-riscv64 \
  -nographic -machine virt  \
  -smp 8 -m 8G \
  -audiodev pa,id=snd0 \
  -kernel fw_payload_oe_qemuvirt.elf \
  -bios none \
  -drive file=mugen_ready.qcow2,format=qcow2,id=hd0 \
  -object rng-random,filename=/dev/urandom,id=rng0 \
  -device virtio-rng-device,rng=rng0 \
  -device virtio-blk-device,drive=hd0 \
  -device virtio-net-device,netdev=usernet \
  -netdev user,id=usernet,hostfwd=tcp::"ssh_port"-:22 \
  -device qemu-xhci -usb -device usb-kbd -device usb-tablet -device usb-audio,audiodev=snd0 \
  -append 'root=/dev/vda1 rw console=ttyS0 swiotlb=1 loglevel=3 systemd.default_timeout_start_sec=600 selinux=0 highres=off mem=8192M earlycon' 
  ```
- 附加硬盘qemu参数：
  
  ```shell
  -drive file=disk"self.id-i".qcow2,format=qcow2,id=hd"i" -device virtio-blk-pci,drive=hd"i"
  ```
  
    self.id：测试时为对应虚拟机的id  
    i：测试时为某一虚拟机的硬盘序号  
- mugen_ready.qcow2处理  
    mugen_ready.qcow2由原始镜像openEuler-22.03-V2-base-qemu-preview.qcow2安装git和mugen依赖而来  
  
  ## 测试结果说明
  
  ### 测试结果文件结构
- [logs文件夹](https://github.com/brsf11/Tarsier-Internship/tree/main/Testing/openEuler-RISC-V-22.03-Preview-V2/logs)：所有测试用例的日志文件  
- [logs_failed文件夹](https://github.com/brsf11/Tarsier-Internship/tree/main/Testing/openEuler-RISC-V-22.03-Preview-V2/logs_failed)：所有未通过测试用例的日志文件  
- [result.csv](https://github.com/brsf11/Tarsier-Internship/blob/main/Testing/openEuler-RISC-V-22.03-Preview-V2/result.csv):测试结果的详细数据统计  
- [failureCause.csv](https://github.com/brsf11/Tarsier-Internship/blob/main/Testing/openEuler-RISC-V-22.03-Preview-V2/failureCause.csv)：测试未通过原因的初步分析和归类  
  
  ### 测试日志说明
- 测试日志中包含测试运行时执行的命令和命令的打印信息，以+（一个或多个）开头的行为执行的命令，不以+开头的行为命令的打印信息  
- 例如假设fio/oe_test_fio_002日志67~87行  
  
  ```
  + fio-dedupe -c 1 /dev/
  + grep 'items processed'
  + CHECK_RESULT 1 0 0 'fio-dedupe -c option failed'
  + actual_result=1
  + expect_result=0
  + mode=0
  + error_log='fio-dedupe -c option failed'
  + '[' -z 1 ']'
  + '[' 0 -eq 0 ']'
  + test 1x '!=' 0x
  + test -n 'fio-dedupe -c option failed'
  + LOG_ERROR 'fio-dedupe -c option failed'
  + message='fio-dedupe -c option failed'
  + python3 /root/mugen-riscv/libs/locallibs/mugen_log.py --level error --message 'fio-dedupe -c option failed'
  Wed Dec  7 01:12:07 2022 - ERROR - fio-dedupe -c option failed
  + (( exec_result++ ))
  + LOG_ERROR 'oe_test_fio_002.sh line 33'
  + message='oe_test_fio_002.sh line 33'
  + python3 /root/mugen-riscv/libs/locallibs/mugen_log.py --level error --message 'oe_test_fio_002.sh line 33'
  Wed Dec  7 01:12:08 2022 - ERROR - oe_test_fio_002.sh line 33
  + return 0
  ```
  
    对应测试用例fio/oe_test_fio_002.sh中32~33行  
  
  ```
  fio-dedupe -c 1 /dev/${local_disk1} | grep "items processed"
  CHECK_RESULT $? 0 0 "fio-dedupe -c option failed"
  ```
  
    测试用例先执行```fio-dedupe -c 1 /dev/${local_disk1} | grep "items processed"```  
    后执行```CHECK_RESULT $? 0 0 "fio-dedupe -c option failed"```比较返回值是否等于0  
    日志```Wed Dec  7 01:12:08 2022 - ERROR - oe_test_fio_002.sh line 33```中提示出错代码对应的用例代码文件为oe_test_fio_002.sh，行数为33行  
  
  ## 测试结果
  
  ### 测试结果说明
- 相对此前的oE自动化测试，本次测试扩展了测试范围，测试了范围内测试套中所有测试用例（不包括描述文件中指定需要多机/网卡资源的用例），导致未通过用例比例比此前几次测试大。  
  
  ### 测试未通过原因分析和归类说明
- 由于未通过测试较多，我们对测试结果进行了初步的分类和归因，具体结果在[failureCause.csv](https://github.com/brsf11/Tarsier-Internship/blob/main/Testing/openEuler-RISC-V-22.03-Preview-V2/failureCause.csv)文件中  
- 测试未通过原因分析和归类由自动化程序完成，基本的原理为对log文件进行字符串匹配  
- 本次测试中，测试未通过原因或者用例未通过原因有以下类型：  
  - 测试用例不能（完全）执行: broken testcase  
  - 软件包缺失: pkg not found  
  - 预装缺失: preinstall absent  
  - 内核模块缺失: kernel module absent  
  - 文件缺失（软件包已安装）: file missing  
  - systemd单元错误  
    - 重启错误: systemd unit restart failure  
    - 运行时错误: systemd unit runtime error  
    - 使能错误: systemd unit enable failure  
  - 超时: timeout  
  - 无效参数: invalid argument  
  - 其他（未被归类）  
- 一个用例可能会有多个类型的测试未通过原因或者未被归类（其他）  
  
  ### 测试未通过原因类型说明
  
  #### 较大概率为软件缺陷造成：
- 文件缺失（软件包已安装）: file missing  
  - 判断标准：用例有执行软件包安装操作，遇到command not found/.service not found/No such file or directory的情况  
  - 说明:可能为安装的软件包中缺少对应文件，软件打包存在问题  
- systemd单元错误  
  - 重启错误: systemd unit restart failure  
    - 用例中测试systemd单元重启操作失败  
  - 运行时错误: systemd unit runtime error  
    - 测试中systemd单元的日志中有报错信息  
  - 使能错误: systemd unit enable failure  
    - 用例中测试systemd单元使能操作失败
- 其他（未被归类）  
  - 其他（未被归类）的测试未通过原因中，大部分为被测软件运行出错，但无法被归为某一特定类型，较大概率为软件bug 
    
    #### 较小概率或并非为软件缺陷造成：
- 超时: timeout  
  - 可能为软件缺陷造成  
  - 判断标准：日志中出现timeout等关键词  
- 测试用例不能（完全）执行: broken testcase  
  - 可能为软件缺陷造成  
  - 判断标准：测试用例没有执行主体测试代码（run_test函数）  
  - 说明:一般为用例运行时在环境准备阶段出错退出  
- 无效参数: invalid argument  
  - 小概率为软件缺陷造成  
  - 判断标准：日志中出现invalid option/invalid argument等关键词  
  - 说明:测试中执行了被测程序的无效参数，可能为用例编写有问题或程序更新后参数变化  
- 软件包缺失: pkg not found  
  - 并非软件缺陷造成  
  - 判断标准：用例运行时软件包安装操作（DNF_INSTALL函数）遇到软件包缺失  
  - 说明:由于测试的测试套都有软件源中对应的软件包，运行测试时遇到的软件包缺失为与测试套对应软件包相关的软件包缺失（如相关的库等）  
- 预装缺失: preinstall absent  
  - 并非为软件缺陷造成  
  - 判断标准：用例没有执行软件包安装操作，遇到command not found/.service not found/No such file or directory的情况    
  - 说明:一般为用例将某些软件包视为预装但被测镜像中并未预装    
- 内核模块缺失: kernel module absent  
  - 并非为软件缺陷造成  
  - 判断标准：modprobe时报错xxx module not found    
  - 说明:缺失对应的内核模块  
