@[TOC]
# 资源
> nuttx_fat32.md

# 通过ramdisk，格式化成fat12文件系统
通过ramdisk讲解FAT文件系统格式
```
--------------
0： FSINFO       
-------------
1： FAT1
--------
2： FAT2 
-------
3 : 根目录
占用： 34-3+1 =32个扇区
这块扇区也很大：
-------
数据扇区 35号 ： 63-35+1= 29个扇区
```

## mkrd -m 1 -s 512 64
> 内存块设备是地址是0x20026380
```
nsh> ls
/:
 dev/
 etc/
nsh> mkrd -m 1 -s 512 64
cmd_mkrd: RAMDISK at 20026380
ramdisk_register: buffer: 20026380 nsectors: 64 sectsize: 512
```

## mkfatfs /dev/ram1
```
nsh> mkfatfs /dev/ram12
rd_open: rd_crefs: 1
rd_geometry: Entry
rd_geometry: available: true mediachanged: false writeenabled: true
rd_geometry: nsectors: 64 sectorsize: 512
mkfatfs_clustersearch: Configuring with 1 sectors/cluster...
mkfatfs_tryfat12: nfatsects=1 nclusters=29 (max=341)
mkfatfs_tryfat16: nfatsects=1 nclusters=29 (min=4081 max=256)
mkfatfs_tryfat16: ERROR: Too few or too many clusters for FAT16: 4081 < 29 < 254
mkfatfs_clustersearch: ERROR: Cannot format FAT16 at 1 sectors/cluster
mkfatfs_selectfat: Selected FAT12
rd_write: sector: 0 nsectors: 512 sectorsize: 1
rd_write: Transfer 512 bytes from 20026380
rd_write: sector: 1 nsectors: 512 sectorsize: 1
rd_write: Transfer 512 bytes from 20026580
rd_write: sector: 2 nsectors: 512 sectorsize: 1
rd_write: Transfer 512 bytes from 20026780
rd_write: sector: 3 nsectors: 512 sectorsize: 1
rd_write: Transfer 512 bytes from 20026980
rd_write: sector: 4 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20026b80                                                                                
rd_write: sector: 5 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20026d80                                                                                
rd_write: sector: 6 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20026f80                                                                                
rd_write: sector: 7 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20027180                                                                                
rd_write: sector: 8 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20027380                                                                                
rd_write: sector: 9 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20027580                                                                                
rd_write: sector: 10 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20027780                                                                                
rd_write: sector: 11 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20027980                                                                                
rd_write: sector: 12 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20027b80                                                                                
rd_write: sector: 13 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20027d80                                                                                
rd_write: sector: 14 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20027f80                                                                                
rd_write: sector: 15 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028180                                                                                
rd_write: sector: 16 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028380                                                                                
rd_write: sector: 17 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028580                                                                                
rd_write: sector: 18 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028780                                                                                
rd_write: sector: 19 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028980                                                                                
rd_write: sector: 20 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028b80                                                                                
rd_write: sector: 21 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028d80                                                                                
rd_write: sector: 22 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20028f80                                                                                
rd_write: sector: 23 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029180                                                                                
rd_write: sector: 24 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029380                                                                                
rd_write: sector: 25 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029580                                                                                
rd_write: sector: 26 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029780                                                                                
rd_write: sector: 27 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029980                                                                                
rd_write: sector: 28 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029b80                                                                                
rd_write: sector: 29 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029d80                                                                                
rd_write: sector: 30 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 20029f80                                                                                
rd_write: sector: 31 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 2002a180                                                                                
rd_write: sector: 32 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 2002a380                                                                                
rd_write: sector: 33 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 2002a580                                                                                
rd_write: sector: 34 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 2002a780                                                                                
rd_close: rd_crefs: 0  
```
###  查看MSB内容，结束符号是0x55aa
```
(gdb) x/128xw 0x20026380 
0x20026380:	0x4e903ceb	0x58545455	0x00202020	0x00010102
0x20026390:	0x40020002	0x0001f800	0x00ff003f	0x00000000
0x200263a0:	0x00000000	0x00290000	0x20000000	0x20202020
0x200263b0:	0x20202020	0x41462020	0x20323154	0x1f0e2020
0x200263c0:	0xac7c5bbe	0x0b74c022	0xbb0eb456	0x10cd0007
0x200263d0:	0x32f0eb5e	0xcd16cde4	0x54feeb19	0x20736968
0x200263e0:	0x6e207369	0x6120746f	0x6f6f6220	0x6c626174
0x200263f0:	0x69642065	0x202e6b73	0x656c5020	0x20657361
0x20026400:	0x65736e69	0x61207472	0x6f6f6220	0x6c626174
0x20026410:	0x6c662065	0x7970706f	0x646e6120	0x72700a0d
0x20026420:	0x20737365	0x20796e61	0x2079656b	0x74206f74
0x20026430:	0x61207972	0x6e696167	0x2e2e2e20	0x00000a0d
0x20026440:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026450:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026460:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026470:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026480:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026490:	0x00000000	0x00000000	0x00000000	0x00000000
0x200264a0:	0x00000000	0x00000000	0x00000000	0x00000000
0x200264b0:	0x00000000	0x00000000	0x00000000	0x00000000
0x200264c0:	0x00000000	0x00000000	0x00000000	0x00000000
0x200264d0:	0x00000000	0x00000000	0x00000000	0x00000000
0x200264e0:	0x00000000	0x00000000	0x00000000	0x00000000
0x200264f0:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026500:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026510:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026520:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026530:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026540:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026550:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026560:	0x00000000	0x00000000	0x00000000	0x00000000
0x20026570:	0x00000000	0x00000000	0x00000000	0xaa550000
```

### FAT1表
```
(gdb) x/128xw 0x20026380+512
0x20026580:	0x00fffff8	0x00000000	0x00000000	0x00000000
0x20026590:	0x00000000	0x00000000	0x00000000	0x00000000
0x200265a0:	0x00000000	0x00000000	0x00000000	0x00000000
```

### FAT2表
```
(gdb) x/32xw 0x20026380+1024
0x20026780:	0x00fffff8	0x00000000	0x00000000	0x00000000
0x20026790:	0x00000000	0x00000000	0x00000000	0x00000000
0x200267a0:	0x00000000	0x00000000	0x00000000	0x00000000
```

## 挂载文件系统nsh> mount -t vfat /dev/ram1 /tmp
```
rd_open: rd_crefs: 1
rd_geometry: Entry
rd_geometry: available: true mediachanged: false writeenabled: true
rd_geometry: nsectors: 64 sectorsize: 512
rd_read: sector: 0 nsectors: 512 sectorsize: 1
rd_read: Transfer 512 bytes from 20026380
fat_mount: FAT12:
fat_mount: 	HW  sector size:     512
fat_mount: 	    sectors:         64
fat_mount: 	FAT reserved:        1
fat_mount: 	    sectors:         64
fat_mount: 	    start sector:    1
fat_mount: 	    root sector:     3
fat_mount: 	    root entries:    512
fat_mount: 	    data sector:     35
fat_mount: 	    FSINFO sector:   0
fat_mount: 	    Num FATs:        2
fat_mount: 	    FAT sectors:     1
fat_mount: 	    sectors/cluster: 1
fat_mount: 	    max clusters:    29
fat_mount: 	FSI free count       -1
fat_mount: 	    next free        0
```

## 写1个文件
> 从日志中分析先读第1个扇区读取FAT表，把内容写道到35号扇区，然后更新FAT1,再更新FAT2, 
>然后写入文件信息到根目录
```
nsh> cd tmp
rd_geometry: Entry
rd_geometry: available: true mediachanged: false writeenabled: true
rd_geometry: nsectors: 64 sectorsize: 512

nsh> echo 111 >> 111.txt
rd_geometry: Entry                                                                                                        
rd_geometry: available: true mediachanged: false writeenabled: true                                                       
rd_geometry: nsectors: 64 sectorsize: 512                                                                                 
rd_geometry: Entry                                                                                                        
rd_geometry: available: true mediachanged: false writeenabled: true                                                       
rd_geometry: nsectors: 64 sectorsize: 512                                                                                 
rd_write: sector: 3 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20026980                                                                                
rd_read: sector: 1 nsectors: 512 sectorsize: 1                                                                            
rd_read: Transfer 512 bytes from 20026580                                                                                 
fat_currentsector: position=0 currentsector=35 sectorsincluster=1                                                         
rd_geometry: Entry                                                                                                        
rd_geometry: available: true mediachanged: false writeenabled: true                                                       
rd_geometry: nsectors: 64 sectorsize: 512                                                                                 
rd_write: sector: 35 nsectors: 512 sectorsize: 1                                                                          
rd_write: Transfer 512 bytes from 2002a980                                                                                
rd_write: sector: 1 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20026580                                                                                
rd_write: sector: 2 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20026780                                                                                
rd_read: sector: 3 nsectors: 512 sectorsize: 1                                                                            
rd_read: Transfer 512 bytes from 20026980                                                                                 
rd_write: sector: 3 nsectors: 512 sectorsize: 1                                                                           
rd_write: Transfer 512 bytes from 20026980 
```

###  根文件信息
>FAT1是第一个扇区
```
(gdb) x/32cb 0x20026980
0x20026980:	49 '1'	49 '1'	49 '1'	32 ' '	32 ' '	32 ' '	32 ' '	32 ' '
0x20026988:	84 'T'	88 'X'	84 'T'	32 ' '	16 '\020'	0 '\000'	-21 '\353'	0 '\000'
0x20026990:	33 '!'	40 '('	0 '\000'	0 '\000'	0 '\000'	0 '\000'	-21 '\353'	0 '\000'
0x20026998:	33 '!'	40 '('	2 '\002'	0 '\000'	5 '\005'	0 '\000'	0 '\000'	0 '\000'
```

### 内容
>内容是35扇区，0x20026380是0号扇区
```
(gdb) x/5cb 0x20026380+35*512
0x2002a980:	49 '1'	49 '1'	49 '1'	32 ' '	10 '\n'
```

### FAT1表
> FAT12占用的12个字节，0号簇： F8 FF FF ， 1号簇： FF FF FF 
> 新文件1111.txt的簇号是 FF FF FF 
```
(gdb) x/32xw 0x20026380+1*512
0x20026580:	0xfffffff8	0x0000000f	0x00000000	0x00000000
```

### FAT2表
> FAT2表和FAT1是一样的
```
(gdb) x/32xw 0x20026380+2*512
0x20026780:	0xfffffff8	0x0000000f	0x00000000	0x00000000
0x20026790:	0x00000000	0x00000000	0x00000000	0x00000000
0x200267a0:	0x00000000	0x00000000	0x00000000	0x00000000
0x200267b0:	0x00000000	0x00000000	0x00000000	0x00000000
```




