# 代码提交

请将本次实验的 **所有代码** 和 **实验报告** 打包提交到作业平台！

# 实验验收指南

> 以下两点二选一即可

1. 完成test.sh的基础功能测试
   
2. 有些同学遇到 **脚本** 无法通过情况，可能是 **FUSE本身的BUG** ，这里同学只要能够进行 **正确的序列演示** 即可，正确的序列如下（基础分30分）：

      - **F5挂载文件系统（1分）**

      - **mkdir测试（4分）**

     ```shell
     mkdir ./tests/mnt/dir0
     mkdir ./tests/mnt/dir0/dir0
     mkdir ./tests/mnt/dir0/dir0/dir0
     mkdir ./tests/mnt/dir1
     ```

      - **touch测试（5分）**

     ```shell
     touch ./tests/mnt/file0
     touch ./tests/mnt/dir0/file0
     touch ./tests/mnt/dir0/dir0/file0
     touch ./tests/mnt/dir0/dir0/dir0/file0
     touch ./tests/mnt/dir1/file0
     ```

      - **ls测试（4分）**

     ```shell
     mkdir ./tests/mnt/dir0
     touch ./tests/mnt/file0
     mkdir ./tests/mnt/dir0/dir1
     mkdir ./tests/mnt/dir0/dir1/dir2
     touch ./tests/mnt/dir0/dir1/dir2/file4
     touch ./tests/mnt/dir0/dir1/dir2/file5
     touch ./tests/mnt/dir0/dir1/dir2/file6
     touch ./tests/mnt/dir0/dir1/file3
     touch ./tests/mnt/dir0/file1
     --------------
     ls ./tests/mnt
     ls ./tests/mnt/dir0
     ls ./tests/mnt/dir0/dir1
     ls ./tests/mnt/dir0/dir1/dir2
     ```
   
      - **remount测试（16分）**
      
     ```console
     fusermount -u ./tests/mnt
     ddriver -r
     # F5再次挂载文件系统
     mkdir ./tests/mnt/hello
     ls ./tests/mnt
     fusermount -u ./tests/mnt
     python3 ./tests/checkbm/checkbm.py -l ./include/fs.layout -r ./tests/checkbm/golden.json
     ```
   
   以上命令均测试成功（**ls输出正常**），则测试通过。

