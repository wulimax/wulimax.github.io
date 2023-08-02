# Kernel oom log分析

 带着问题学习为什么会OOM

```tex
Aug  1 10:05:31 localhost kernel: java invoked oom-killer: gfp_mask=0x201da, order=0, oom_score_adj=0

#----------------------------注释start
java程序申请了内存4k内存
order是个数组每个占4K大小,order=0也就是4k
#----------------------------注释end

Aug  1 10:05:31 localhost kernel: java cpuset=/ mems_allowed=0
Aug  1 10:05:31 localhost kernel: CPU: 7 PID: 18524 Comm: java Not tainted 3.10.0-862.el7.x86_64 #1
Aug  1 10:05:31 localhost kernel: Hardware name: VMware, Inc. VMware Virtual Platform/440BX Desktop Reference Platform, BIOS 6.00 07/03/2018
Aug  1 10:05:31 localhost kernel: Call Trace:
Aug  1 10:05:31 localhost kernel: [<ffffffffb9b0d768>] dump_stack+0x19/0x1b
Aug  1 10:05:31 localhost kernel: [<ffffffffb9b090ea>] dump_header+0x90/0x229
Aug  1 10:05:31 localhost kernel: [<ffffffffb94f76e2>] ? ktime_get_ts64+0x52/0xf0
Aug  1 10:05:31 localhost kernel: [<ffffffffb954e0bf>] ? delayacct_end+0x8f/0xb0
Aug  1 10:05:31 localhost kernel: [<ffffffffb9597904>] oom_kill_process+0x254/0x3d0
Aug  1 10:05:31 localhost kernel: [<ffffffffb95973ad>] ? oom_unkillable_task+0xcd/0x120
Aug  1 10:05:31 localhost kernel: [<ffffffffb9597456>] ? find_lock_task_mm+0x56/0xc0
Aug  1 10:05:31 localhost kernel: [<ffffffffb9598146>] out_of_memory+0x4b6/0x4f0
Aug  1 10:05:31 localhost kernel: [<ffffffffb959ea64>] __alloc_pages_nodemask+0xaa4/0xbb0
Aug  1 10:05:31 localhost kernel: [<ffffffffb95e8528>] alloc_pages_current+0x98/0x110
Aug  1 10:05:31 localhost kernel: [<ffffffffb9593cf7>] __page_cache_alloc+0x97/0xb0
Aug  1 10:05:31 localhost kernel: [<ffffffffb95963f8>] filemap_fault+0x298/0x490
Aug  1 10:05:31 localhost kernel: [<ffffffffc04c1e66>] ext4_filemap_fault+0x36/0x50 [ext4]
Aug  1 10:05:31 localhost kernel: [<ffffffffb95c05fa>] __do_fault.isra.58+0x8a/0x100
Aug  1 10:05:31 localhost kernel: [<ffffffffb95c0b5c>] do_read_fault.isra.60+0x4c/0x1a0
Aug  1 10:05:31 localhost kernel: [<ffffffffb95c52dc>] handle_pte_fault+0x2dc/0xc30
Aug  1 10:05:31 localhost kernel: [<ffffffffb9503534>] ? futex_wait_queue_me+0xa4/0x130
Aug  1 10:05:31 localhost kernel: [<ffffffffb95c747d>] handle_mm_fault+0x39d/0x9b0
Aug  1 10:05:31 localhost kernel: [<ffffffffb9b1a597>] __do_page_fault+0x197/0x4f0
Aug  1 10:05:31 localhost kernel: [<ffffffffb9b1a925>] do_page_fault+0x35/0x90
Aug  1 10:05:31 localhost kernel: [<ffffffffb9b16768>] page_fault+0x28/0x30
Aug  1 10:05:31 localhost kernel: Mem-Info:

Aug  1 10:05:31 localhost kernel: active_anon:3001240 inactive_anon:445642 isolated_anon:0#012 active_file:286 inactive_file:1 isolated_file:0#012 unevictable:0 dirty:0 writeback:400 unstable:0#012 slab_reclaimable:15633 slab_unreclaimable:17532#012 mapped:5056 shmem:81416 pagetables:10972 bounce:0#012 free:45962 free_pcp:209 free_cma:0
Aug  1 10:05:31 localhost kernel: Node 0 DMA free:15892kB min:36kB low:44kB high:52kB active_anon:0kB inactive_anon:0kB active_file:0kB inactive_file:0kB unevictable:0kB isolated(anon):0kB isolated(file):0kB present:15992kB managed:15908kB mlocked:0kB dirty:0kB writeback:0kB mapped:0kB shmem:0kB slab_reclaimable:0kB slab_unreclaimable:16kB kernel_stack:0kB pagetables:0kB unstable:0kB bounce:0kB free_pcp:0kB local_pcp:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:0 all_unreclaimable? yes
Aug  1 10:05:31 localhost kernel: lowmem_reserve[]: 0 2991 28122 28122
Aug  1 10:05:31 localhost kernel: Node 0 DMA32 free:107652kB min:7184kB low:8980kB high:10776kB active_anon:1233752kB inactive_anon:490708kB active_file:220kB inactive_file:0kB unevictable:0kB isolated(anon):0kB isolated(file):0kB present:3129216kB managed:3063648kB mlocked:0kB dirty:0kB writeback:0kB mapped:2456kB shmem:70156kB slab_reclaimable:8904kB slab_unreclaimable:6444kB kernel_stack:3392kB pagetables:5496kB unstable:0kB bounce:0kB free_pcp:384kB local_pcp:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:854 all_unreclaimable? yes
Aug  1 10:05:31 localhost kernel: lowmem_reserve[]: 0 0 25130 25130
Aug  1 10:05:31 localhost kernel: Node 0 Normal free:60304kB min:60360kB low:75448kB high:90540kB active_anon:10771208kB inactive_anon:1291860kB active_file:924kB inactive_file:612kB unevictable:0kB isolated(anon):0kB isolated(file):0kB present:26214400kB managed:25736672kB mlocked:0kB dirty:0kB writeback:1600kB mapped:17768kB shmem:255508kB slab_reclaimable:53628kB slab_unreclaimable:63668kB kernel_stack:24496kB pagetables:38392kB unstable:0kB bounce:0kB free_pcp:452kB local_pcp:0kB free_cma:0kB writeback_tmp:0kB pages_scanned:3200 all_unreclaimable? yes
Aug  1 10:05:31 localhost kernel: lowmem_reserve[]: 0 0 0 0
Aug  1 10:05:31 localhost kernel: Node 0 DMA: 1*4kB (U) 0*8kB 1*16kB (U) 0*32kB 2*64kB (U) 1*128kB (U) 1*256kB (U) 0*512kB 1*1024kB (U) 1*2048kB (M) 3*4096kB (M) = 15892kB
Aug  1 10:05:31 localhost kernel: Node 0 DMA32: 4176*4kB (UEM) 1375*8kB (UEM) 1328*16kB (UEM) 392*32kB (UEM) 88*64kB (EM) 23*128kB (UEM) 7*256kB (UEM) 1*512kB (E) 35*1024kB (UE) 0*2048kB 0*4096kB = 108216kB
Aug  1 10:05:31 localhost kernel: Node 0 Normal: 14811*4kB (UM) 2*8kB (M) 0*16kB 0*32kB 0*64kB 0*128kB 0*256kB 0*512kB 0*1024kB 0*2048kB 0*4096kB = 59260kB
Aug  1 10:05:31 localhost kernel: Node 0 hugepages_total=0 hugepages_free=0 hugepages_surp=0 hugepages_size=1048576kB
Aug  1 10:05:31 localhost kernel: Node 0 hugepages_total=0 hugepages_free=0 hugepages_surp=0 hugepages_size=2048kB
Aug  1 10:05:31 localhost kernel: 149383 total pagecache pages
Aug  1 10:05:31 localhost kernel: 67925 pages in swap cache
Aug  1 10:05:31 localhost kernel: Swap cache stats: add 8531221, delete 8462136, find 94044263/94448603
Aug  1 10:05:31 localhost kernel: Free swap  = 0kB
Aug  1 10:05:31 localhost kernel: Total swap = 4063228kB
Aug  1 10:05:31 localhost kernel: 7339902 pages RAM
Aug  1 10:05:31 localhost kernel: 0 pages HighMem/MovableOnly
Aug  1 10:05:31 localhost kernel: 135845 pages reserved
Aug  1 10:05:31 localhost kernel: [ pid ]   uid  tgid total_vm      rss nr_ptes swapents oom_score_adj name
Aug  1 10:05:31 localhost kernel: [  756]     0   756    49632        0      29      113             0 lvmetad
Aug  1 10:05:31 localhost kernel: [ 1414]     0  1414     4844        0      14       50             0 rpc.idmapd
Aug  1 10:05:31 localhost kernel: [ 1415]     0  1415    13881       26      27       87         -1000 auditd
Aug  1 10:05:31 localhost kernel: [ 1439]   999  1439   134615       83      60     1546             0 polkitd
Aug  1 10:05:31 localhost kernel: [ 1440]     0  1440     5421       43      17       48             0 irqbalance
Aug  1 10:05:31 localhost kernel: [ 1443]    32  1443    17309       18      38      130             0 rpcbind
Aug  1 10:05:31 localhost kernel: [ 1446]    81  1446    14556       84      33       81          -900 dbus-daemon
Aug  1 10:05:31 localhost kernel: [ 1450]   997  1450    29441        7      30      108             0 chronyd
Aug  1 10:05:31 localhost kernel: [ 1456]     0  1456    48777        1      35      130             0 gssproxy
Aug  1 10:05:31 localhost kernel: [ 1465]     0  1465    24922        0      43      401             0 VGAuthService
Aug  1 10:05:31 localhost kernel: [ 1466]     0  1466    80024       84      67      282             0 vmtoolsd
Aug  1 10:05:31 localhost kernel: [ 1467]     0  1467   119051      135      81      376             0 NetworkManager
Aug  1 10:05:31 localhost kernel: [ 1713]     0  1713    28205       26      59      232         -1000 sshd
Aug  1 10:05:31 localhost kernel: [ 1715]     0  1715   143466      120      96     2633             0 tuned
Aug  1 10:05:31 localhost kernel: [ 1719]     0  1719     6261        0      17       59             0 xinetd
Aug  1 10:05:31 localhost kernel: [ 1720]     0  1720   184040     3224     171      156             0 rsyslogd
Aug  1 10:05:31 localhost kernel: [ 1725]    29  1725    10609        1      26      208             0 rpc.statd
Aug  1 10:05:31 localhost kernel: [ 1734]     0  1734     6477        1      18       51             0 atd
Aug  1 10:05:31 localhost kernel: [ 1739]     0  1739    27524        2      10       31             0 agetty
Aug  1 10:05:31 localhost kernel: [ 1740]     0  1740    31572       28      18      127             0 crond
Aug  1 10:05:31 localhost kernel: [ 1742]   998  1742    21465       12      42      202             0 zabbix_agentd
Aug  1 10:05:31 localhost kernel: [ 1743]   998  1743    21529      442      43      195             0 zabbix_agentd
Aug  1 10:05:31 localhost kernel: [ 1744]   998  1744    21564       71      43      211             0 zabbix_agentd
Aug  1 10:05:31 localhost kernel: [ 1747]   998  1747    21560       72      44      211             0 zabbix_agentd
Aug  1 10:05:31 localhost kernel: [ 1750]   998  1750    21517       69      43      214             0 zabbix_agentd
Aug  1 10:05:31 localhost kernel: [ 1751]   998  1751    21465       23      43      224             0 zabbix_agentd
Aug  1 10:05:31 localhost kernel: [ 1756]     0  1756    10670        2      24      212             0 rpc.mountd
Aug  1 10:05:31 localhost kernel: [ 2071]     0  2071    22410       20      42      240             0 master
Aug  1 10:05:31 localhost kernel: [ 2079]    89  2079    22480       29      44      238             0 qmgr
Aug  1 10:05:31 localhost kernel: [18422]  1001 18422  3823405   292196    1436   210771             0 java
Aug  1 10:05:31 localhost kernel: [23673]  1001 23673  4361806   423427    1517   169247             0 java
Aug  1 10:05:31 localhost kernel: [32696]  1001 32696  6177638  1941269    4737   264181             0 java
Aug  1 10:05:31 localhost kernel: [29128]  1001 29128  4391694   620535    1688    72863             0 java
Aug  1 10:05:31 localhost kernel: [22734]    89 22734    22473      253      46        0             0 cleanup
Aug  1 10:05:31 localhost kernel: [22736]    89 22736    22437      251      44        0             0 trivial-rewrite
Aug  1 10:05:31 localhost kernel: [22737]     0 22737    22462      265      47        0             0 local
Aug  1 10:05:31 localhost kernel: [22738]    89 22738    22445      249      43        0             0 bounce
Aug  1 10:05:31 localhost kernel: [27888]    89 27888    22436      250      45        0             0 pickup
Aug  1 10:05:31 localhost kernel: [27986]   998 27986    21564       39      40      211             0 zabbix_agentd
Aug  1 10:05:31 localhost kernel: Out of memory: Kill process 32696 (java) score 268 or sacrifice child
Aug  1 10:05:31 localhost kernel: Killed process 32696 (java) total-vm:24710552kB, anon-rss:7765076kB, file-rss:0kB, shmem-rss:0kB
#--------------------------
杀掉了32696进程

```

  反思原因:  java服务设置的-Xmx 加起来大于实际物理内存导致触发了kernel 

