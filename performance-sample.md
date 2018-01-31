Notes
===

These benchs will show performance on small objects. Large object backup speed is limited by Bareos, and are mostly the same as in "classic" backups. You will sature a CPU core and that's it, no option on this plugin will help that. Anyway, a simple bench on large files in included below, for information.

Hardware description
---

Tests are made on a single vm, which hosts a mysql database, bareos-dir, bareos-fd and bareos-sd.  
Tests are made against a Ceph's rgw instance, located nearby (same datacenter).  

VM's specs:  
- 16 CPU core (Xeon 2687v3)
- 32GB of DDR4 memory
- 10Gbps NIC


Dataset: few large files
---

Dataset:
```
1073741824   3
```

Initial backup, using default options:
```
  Elapsed time:           1 min 40 secs
  Priority:               10
  FD Files Written:       3
  SD Files Written:       3
  FD Bytes Written:       3,221,225,472 (3.221 GB)
  SD Bytes Written:       3,221,226,458 (3.221 GB)
  Rate:                   32212.3 KB/s
```

Incremental backup, using default options, with no modified file, with accurate mode:
```
  Elapsed time:           2 secs
  Priority:               10
  FD Files Written:       0
  SD Files Written:       0
  FD Bytes Written:       0 (0 B)
  SD Bytes Written:       0 (0 B)
  Rate:                   0.0 KB/s
```

Incremental backup, using default options, with no modified file, without accurate mode:
```
  Elapsed time:           2 secs
  Priority:               10
  FD Files Written:       0
  SD Files Written:       0
  FD Bytes Written:       0 (0 B)
  SD Bytes Written:       0 (0 B)
  Rate:                   0.0 KB/s
```

Dataset: few small files
---

Dataset (~30k files):
```
         0 105
         1   2
         2   5
         4  29
         8  44
        16 104
        32 426
        64 1292
       128 3160
       256 4086
       512 5450
      1024 5082
      2048 4515
      4096 2840
      8192 1463
     16384 642
     32768 215
     65536  62
    131072  34
    262144  20
    524288  17
   1048576  11
   2097152   5
   4194304   7
   8388608   2
  16777216   3
  33554432   4
  67108864   2
 268435456   1
```

Initial backup, using default options:
```
  Elapsed time:           1 min 30 secs
  Priority:               10
  FD Files Written:       29,626
  SD Files Written:       29,626
  FD Bytes Written:       1,176,889,787 (1.176 GB)
  SD Bytes Written:       1,188,585,775 (1.188 GB)
  Rate:                   13076.6 KB/s
```

Incremental backup, using default options, with no modified file, without accurate mode:
```
  Elapsed time:           13 secs
  Priority:               10
  FD Files Written:       0
  SD Files Written:       0
  FD Bytes Written:       0 (0 B)
  SD Bytes Written:       0 (0 B)
  Rate:                   0.0 KB/s
```

Incremental backup, using default options, with no modified file, with accurate mode:
```
  Elapsed time:           19 secs
  Priority:               10
  FD Files Written:       0
  SD Files Written:       0
  FD Bytes Written:       0 (0 B)
  SD Bytes Written:       0 (0 B)
  Rate:                   0.0 KB/s
```

Initial backup, using 256 prefetchers:
```
  Elapsed time:           1 min 18 secs
  Priority:               10
  FD Files Written:       29,626
  SD Files Written:       29,626
  FD Bytes Written:       1,176,889,787 (1.176 GB)
  SD Bytes Written:       1,189,119,043 (1.189 GB)
  Rate:                   15088.3 KB/s
```

Initial backup, using 768 prefetchers:
```
  Elapsed time:           1 min 53 secs
  Priority:               10
  FD Files Written:       29,626
  SD Files Written:       29,626
  FD Bytes Written:       1,176,889,787 (1.176 GB)
  SD Bytes Written:       1,189,119,043 (1.189 GB)
  Rate:                   10415.0 KB/s
```
The latter takes extra time to spawn all the prefetcher, without benefits due to the small dataset.


Dataset: many small files
---

Dataset (1.3M files):
```
         0 4447
         1  84
         2 210
         4 1218
         8 1856
        16 4399
        32 42269
        64 54950
       128 137627
       256 175792
       512 232435
      1024 219163
      2048 195773
      4096 126805
      8192 68979
     16384 28939
     32768 10646
     65536 3775
    131072 7584
    262144 3459
    524288 970
   1048576 485
   2097152 221
   4194304 301
   8388608  85
  16777216 133
  33554432 171
  67108864  86
 268435456  42

```

Initial backup, using default options:
```
  Elapsed time:           1 hour 6 mins 39 secs
  Priority:               10
  FD Files Written:       1,322,818
  SD Files Written:       1,322,818
  FD Bytes Written:       52,679,708,301 (52.67 GB)
  SD Bytes Written:       53,217,728,881 (53.21 GB)
  Rate:                   13173.2 KB/s
```

Incremental backup, using default options, with no modified file, without accurate mode:
```
  Elapsed time:           9 mins 17 secs
  Priority:               10
  FD Files Written:       0
  SD Files Written:       0
  FD Bytes Written:       0 (0 B)
  SD Bytes Written:       0 (0 B)
  Rate:                   0.0 KB/s
```

Incremental backup, using default options, with no modified file, with accurate mode:
```
  Elapsed time:           15 mins 38 secs
  Priority:               10
  FD Files Written:       86
  SD Files Written:       86
  FD Bytes Written:       0 (0 B)
  SD Bytes Written:       34,163 (34.16 KB)
  Rate:                   0.0 KB/s
```

Initial backup, using 256 prefetchers:
```
  Elapsed time:           52 mins 39 secs
  Priority:               10
  FD Files Written:       1,322,818
  SD Files Written:       1,322,818
  FD Bytes Written:       52,679,708,301 (52.67 GB)
  SD Bytes Written:       53,241,539,605 (53.24 GB)
  Rate:                   16676.1 KB/s
```

Initial backup, using 768 prefetchers:
```
  Elapsed time:           51 mins 26 secs
  Priority:               10
  FD Files Written:       1,322,818
  SD Files Written:       1,322,818
  FD Bytes Written:       52,679,708,301 (52.67 GB)
  SD Bytes Written:       53,241,539,605 (53.24 GB)
  Rate:                   17070.5 KB/s
```

Backuping the same dataset locally (without the plugin, data is store on a regular filesystem):
```
  Elapsed time:           25 mins 8 secs
  Priority:               10
  FD Files Written:       1,629,535
  SD Files Written:       1,629,535
  FD Bytes Written:       52,679,708,301 (52.67 GB)
  SD Bytes Written:       52,950,563,974 (52.95 GB)
  Rate:                   34933.5 KB/s
```
