# 自动化测试报告(Part 3)

 本次测试为最新版本的oE2203v2的自动化测试报告，本部分涵盖了大约60个测试套。

### 测试步骤：

1. 启动openEuler环境，打开mugen文件夹
2. 安装依赖 `python3` `python-paramiko`
3. 在 lists 文件夹新建待测试套件的集合的文件
4. 执行命令：`sudo python3 qemu_test.py -w /home/yx/Desktop/openEuler/ -B none -K fw_payload_oe_qemuvirt.elf -D openEuler-22.03-V1-riscv64-qemu-xfce.qcow2 -x 2 -c 8 -M 8 -g -l lists/part3`
5. 等待运行完成即可

### 测试结果：

- logs：包含所有测试用例的日志文件
- logs_failed：包含所有未通过测试用例的日志文件
- suite2cases_out：包含json格式的结果输出的文件
- exec_log：执行时的输出日志


