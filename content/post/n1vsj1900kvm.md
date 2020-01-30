---
title: "使用unixbench测试N1的性能"
date: 2020-01-11T18:23:51+08:00
draft: false
---

由于n1是arm64架构，无法使用geekbench等程序进行测试，而sysbench等程序测出来等结果复杂度并无法让我满意，所以使用unixbench进行了测试。  

测试脚本来自:[http://www.unixbench.org/](http://www.unixbench.org/)  

```shell
wget --no-check-certificate https://github.com/zq/unixbench/raw/master/unixbench.sh
chmod +x unixbench.sh
./unixbench.sh
```

系统是armbian所以需要修改一下shell里的系统判断逻辑。测试完成后结果如下：

```shell

========================================================================
   BYTE UNIX Benchmarks (Version 5.1.3)

   System: armbian: GNU/Linux
   OS: GNU/Linux -- 5.4.10-amlogic-flippy-21+ -- #19 SMP PREEMPT Fri Jan 10 00:50:09 CST 2020
   Machine: aarch64 (unknown)
   Language: en_US.utf8 (charmap="UTF-8", collate="UTF-8")
   CPU 0: ARMv8 Processor rev 4 (v8l) (0.0 bogomips)

   CPU 1: ARMv8 Processor rev 4 (v8l) (0.0 bogomips)

   CPU 2: ARMv8 Processor rev 4 (v8l) (0.0 bogomips)

   CPU 3: ARMv8 Processor rev 4 (v8l) (0.0 bogomips)

   17:23:10 up 22 min,  1 user,  load average: 0.35, 0.26, 0.17; runlevel 5

------------------------------------------------------------------------
Benchmark Run: Sat Jan 11 2020 17:23:10 - 17:51:09
4 CPUs in system; running 1 parallel copy of tests

Dhrystone 2 using register variables        7114792.2 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     1654.7 MWIPS (9.8 s, 7 samples)
Execl Throughput                                903.4 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        173178.2 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           51129.5 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        456448.6 KBps  (30.0 s, 2 samples)
Pipe Throughput                              358258.1 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                  71258.5 lps   (10.0 s, 7 samples)
Process Creation                               2097.3 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   2427.1 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    709.1 lpm   (60.0 s, 2 samples)
System Call Overhead                         670984.2 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0    7114792.2    609.7
Double-Precision Whetstone                       55.0       1654.7    300.9
Execl Throughput                                 43.0        903.4    210.1
File Copy 1024 bufsize 2000 maxblocks          3960.0     173178.2    437.3
File Copy 256 bufsize 500 maxblocks            1655.0      51129.5    308.9
File Copy 4096 bufsize 8000 maxblocks          5800.0     456448.6    787.0
Pipe Throughput                               12440.0     358258.1    288.0
Pipe-based Context Switching                   4000.0      71258.5    178.1
Process Creation                                126.0       2097.3    166.5
Shell Scripts (1 concurrent)                     42.4       2427.1    572.4
Shell Scripts (8 concurrent)                      6.0        709.1   1181.8
System Call Overhead                          15000.0     670984.2    447.3
                                                                   ========
System Benchmarks Index Score                                         384.9

------------------------------------------------------------------------
Benchmark Run: Sat Jan 11 2020 17:51:09 - 18:19:06
4 CPUs in system; running 4 parallel copies of tests

Dhrystone 2 using register variables       28346796.3 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     6618.6 MWIPS (9.8 s, 7 samples)
Execl Throughput                               3044.2 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        344872.6 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           94511.9 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        969487.3 KBps  (30.0 s, 2 samples)
Pipe Throughput                             1427182.4 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 236574.2 lps   (10.0 s, 7 samples)
Process Creation                               5388.8 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   5683.4 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    761.4 lpm   (60.2 s, 2 samples)
System Call Overhead                        2581430.3 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   28346796.3   2429.0
Double-Precision Whetstone                       55.0       6618.6   1203.4
Execl Throughput                                 43.0       3044.2    708.0
File Copy 1024 bufsize 2000 maxblocks          3960.0     344872.6    870.9
File Copy 256 bufsize 500 maxblocks            1655.0      94511.9    571.1
File Copy 4096 bufsize 8000 maxblocks          5800.0     969487.3   1671.5
Pipe Throughput                               12440.0    1427182.4   1147.3
Pipe-based Context Switching                   4000.0     236574.2    591.4
Process Creation                                126.0       5388.8    427.7
Shell Scripts (1 concurrent)                     42.4       5683.4   1340.4
Shell Scripts (8 concurrent)                      6.0        761.4   1269.1
System Call Overhead                          15000.0    2581430.3   1721.0
                                                                   ========
System Benchmarks Index Score                                        1032.1
```

单核384分，多核1032分，作为对比，我使用j1900的蜗牛黑群晖里kvm虚拟了一个4核心/768M内存的debian，结果如下：  

```shell

========================================================================
   BYTE UNIX Benchmarks (Version 5.1.3)

   System: debian: GNU/Linux
   OS: GNU/Linux -- 4.19.0-6-amd64 -- #1 SMP Debian 4.19.67-2+deb10u2 (2019-11-11)
   Machine: x86_64 (unknown)
   Language: en_US.utf8 (charmap="UTF-8", collate="UTF-8")
   CPU 0: Intel(R) Celeron(R) CPU J1900 @ 1.99GHz (4000.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET
   CPU 1: Intel(R) Celeron(R) CPU J1900 @ 1.99GHz (4000.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET
   CPU 2: Intel(R) Celeron(R) CPU J1900 @ 1.99GHz (4000.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET
   CPU 3: Intel(R) Celeron(R) CPU J1900 @ 1.99GHz (4000.0 bogomips)
          Hyper-Threading, x86-64, MMX, Physical Address Ext, SYSENTER/SYSEXIT, SYSCALL/SYSRET
   02:24:19 up  1:30,  1 user,  load average: 0.66, 0.19, 0.06; runlevel 5

------------------------------------------------------------------------
Benchmark Run: Sat Jan 11 2020 02:24:19 - 02:52:12
4 CPUs in system; running 1 parallel copy of tests

Dhrystone 2 using register variables       12068303.2 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     2308.9 MWIPS (9.8 s, 7 samples)
Execl Throughput                               1072.5 lps   (29.9 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks         91038.9 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           23638.7 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        298057.5 KBps  (30.0 s, 2 samples)
Pipe Throughput                              128456.4 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                  13256.9 lps   (10.0 s, 7 samples)
Process Creation                               1956.8 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   2373.8 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    560.8 lpm   (60.1 s, 2 samples)
System Call Overhead                         103456.3 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   12068303.2   1034.1
Double-Precision Whetstone                       55.0       2308.9    419.8
Execl Throughput                                 43.0       1072.5    249.4
File Copy 1024 bufsize 2000 maxblocks          3960.0      91038.9    229.9
File Copy 256 bufsize 500 maxblocks            1655.0      23638.7    142.8
File Copy 4096 bufsize 8000 maxblocks          5800.0     298057.5    513.9
Pipe Throughput                               12440.0     128456.4    103.3
Pipe-based Context Switching                   4000.0      13256.9     33.1
Process Creation                                126.0       1956.8    155.3
Shell Scripts (1 concurrent)                     42.4       2373.8    559.9
Shell Scripts (8 concurrent)                      6.0        560.8    934.7
System Call Overhead                          15000.0     103456.3     69.0
                                                                   ========
System Benchmarks Index Score                                         239.2

------------------------------------------------------------------------
Benchmark Run: Sat Jan 11 2020 02:52:12 - 03:20:21
4 CPUs in system; running 4 parallel copies of tests

Dhrystone 2 using register variables       40360483.5 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     8067.7 MWIPS (9.5 s, 7 samples)
Execl Throughput                               2621.0 lps   (29.9 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        181072.3 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks           50847.5 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks        529403.0 KBps  (30.0 s, 2 samples)
Pipe Throughput                              437487.4 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 102733.9 lps   (10.0 s, 7 samples)
Process Creation                               2284.0 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   4443.5 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    761.2 lpm   (60.2 s, 2 samples)
System Call Overhead                         343362.3 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   40360483.5   3458.5
Double-Precision Whetstone                       55.0       8067.7   1466.9
Execl Throughput                                 43.0       2621.0    609.5
File Copy 1024 bufsize 2000 maxblocks          3960.0     181072.3    457.3
File Copy 256 bufsize 500 maxblocks            1655.0      50847.5    307.2
File Copy 4096 bufsize 8000 maxblocks          5800.0     529403.0    912.8
Pipe Throughput                               12440.0     437487.4    351.7
Pipe-based Context Switching                   4000.0     102733.9    256.8
Process Creation                                126.0       2284.0    181.3
Shell Scripts (1 concurrent)                     42.4       4443.5   1048.0
Shell Scripts (8 concurrent)                      6.0        761.2   1268.6
System Call Overhead                          15000.0     343362.3    228.9
                                                                   ========
System Benchmarks Index Score                                         595.2
```

结果很让人惊讶的，N1的性能还是很强的。  
