# 自动化测试报告(Part 3)

 本次测试为oE2203v2的自动化测试报告，本部分涵盖了大约60个测试套。

### 测试步骤：

1.  启动openEuler环境，打开mugen文件夹
2.  安装依赖 `python3` `python-paramiko`
3. 在 lists 文件夹新建待测试套件的集合的文件
4. 执行命令：python3 qemu_test.py lists/new_file
5. 等待运行完成即可

### 测试结果：

- logs：包含所有测试用例的日志文件
- logs_failed：包含所有未通过测试用例的日志文件
- suite2cases：包含json格式的结果输出的文件
- exec.log：执行时的输出日志

