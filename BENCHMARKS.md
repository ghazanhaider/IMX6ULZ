# FIO storage


Fio command to compare
```
# fio --loops=5 --size=1000m --filename=/mnt/fs/fiotest.tmp --stonewall --ioengine=libaio --direct=1   --name=Seqread --bs=1m --rw=read   --name=Seqwrite --bs=1m --rw=write   --name=512Kread --bs=512k --rw=randread   --name=512Kwrite --bs=512k --rw=randwrite   --name=4kQD32read --bs=4k --iodepth=32 --rw=randread   --name=4kQD32write --bs=4k --iodepth=32 --rw=randwrite
Seqread: (g=0): rw=read, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=1
Seqwrite: (g=1): rw=write, bs=(R) 1024KiB-1024KiB, (W) 1024KiB-1024KiB, (T) 1024KiB-1024KiB, ioengine=libaio, iodepth=1
512Kread: (g=2): rw=randread, bs=(R) 512KiB-512KiB, (W) 512KiB-512KiB, (T) 512KiB-512KiB, ioengine=libaio, iodepth=1
512Kwrite: (g=3): rw=randwrite, bs=(R) 512KiB-512KiB, (W) 512KiB-512KiB, (T) 512KiB-512KiB, ioengine=libaio, iodepth=1
4kQD32read: (g=4): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
4kQD32write: (g=5): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=32
fio-3.33
Starting 6 processes
Jobs: 1 (f=1): [_(5),w(1)][100.0%][w=8590KiB/s][w=2147 IOPS][eta 00m:00s]
Seqread: (groupid=0, jobs=1): err= 0: pid=18519: Mon Nov 25 23:09:16 2024
  read: IOPS=32, BW=32.8MiB/s (34.4MB/s)(5000MiB/152506msec)
    slat (usec): min=1060, max=31335, avg=16215.66, stdev=2651.00
    clat (usec): min=5754, max=31636, avg=14159.63, stdev=2326.54
     lat (usec): min=24151, max=44922, avg=30375.29, stdev=1633.05
    clat percentiles (usec):
     |  1.00th=[11076],  5.00th=[11994], 10.00th=[12387], 20.00th=[12780],
     | 30.00th=[13173], 40.00th=[13698], 50.00th=[13960], 60.00th=[14222],
     | 70.00th=[14615], 80.00th=[15008], 90.00th=[15270], 95.00th=[15533],
     | 99.00th=[27395], 99.50th=[27919], 99.90th=[28967], 99.95th=[30016],
     | 99.99th=[31589]
   bw (  KiB/s): min=32062, max=36864, per=100.00%, avg=33599.76, stdev=1124.77, samples=300
   iops        : min=   31, max=   36, avg=32.22, stdev= 1.11, samples=300
  lat (msec)   : 10=0.20%, 20=97.28%, 50=2.52%
  cpu          : usr=0.66%, sys=11.05%, ctx=14497, majf=0, minf=278
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=5000,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1
Seqwrite: (groupid=1, jobs=1): err= 0: pid=18533: Mon Nov 25 23:09:16 2024
  write: IOPS=29, BW=29.5MiB/s (30.9MB/s)(5000MiB/169756msec); 0 zone resets
    slat (msec): min=2, max=657, avg=17.78, stdev=13.49
    clat (msec): min=8, max=668, avg=16.04, stdev=10.52
     lat (msec): min=26, max=687, avg=33.82, stdev=16.58
    clat percentiles (msec):
     |  1.00th=[   12],  5.00th=[   13], 10.00th=[   13], 20.00th=[   14],
     | 30.00th=[   15], 40.00th=[   15], 50.00th=[   15], 60.00th=[   16],
     | 70.00th=[   16], 80.00th=[   16], 90.00th=[   23], 95.00th=[   29],
     | 99.00th=[   31], 99.50th=[   37], 99.90th=[   44], 99.95th=[   95],
     | 99.99th=[  667]
   bw (  KiB/s): min= 8031, max=32768, per=100.00%, avg=30276.86, stdev=2457.51, samples=333
   iops        : min=    7, max=   32, avg=28.94, stdev= 2.43, samples=333
  lat (msec)   : 10=0.04%, 20=88.04%, 50=11.84%, 100=0.04%, 250=0.02%
  lat (msec)   : 750=0.02%
  cpu          : usr=2.18%, sys=9.36%, ctx=17211, majf=0, minf=23
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,5000,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1
512Kread: (groupid=2, jobs=1): err= 0: pid=18535: Mon Nov 25 23:09:16 2024
  read: IOPS=61, BW=30.9MiB/s (32.4MB/s)(5000MiB/161608msec)
    slat (usec): min=528, max=8588, avg=1516.86, stdev=775.65
    clat (usec): min=10995, max=25831, avg=14502.08, stdev=1244.77
     lat (usec): min=12474, max=29661, avg=16018.93, stdev=1556.73
    clat percentiles (usec):
     |  1.00th=[12125],  5.00th=[12387], 10.00th=[12649], 20.00th=[13304],
     | 30.00th=[14091], 40.00th=[14484], 50.00th=[14615], 60.00th=[14615],
     | 70.00th=[14746], 80.00th=[15533], 90.00th=[16057], 95.00th=[16712],
     | 99.00th=[17433], 99.50th=[17957], 99.90th=[19792], 99.95th=[20841],
     | 99.99th=[25560]
   bw (  KiB/s): min=29170, max=34608, per=100.00%, avg=31711.86, stdev=1153.45, samples=318
   iops        : min=   56, max=   67, avg=61.48, stdev= 2.24, samples=318
  lat (msec)   : 20=99.93%, 50=0.07%
  cpu          : usr=1.07%, sys=9.26%, ctx=14930, majf=0, minf=145
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=10000,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1
512Kwrite: (groupid=3, jobs=1): err= 0: pid=18537: Mon Nov 25 23:09:16 2024
  write: IOPS=53, BW=26.7MiB/s (28.0MB/s)(5000MiB/187404msec); 0 zone resets
    slat (usec): min=723, max=18368, avg=1710.75, stdev=657.32
    clat (msec): min=10, max=754, avg=16.89, stdev=27.81
     lat (msec): min=12, max=755, avg=18.60, stdev=27.82
    clat percentiles (msec):
     |  1.00th=[   13],  5.00th=[   13], 10.00th=[   13], 20.00th=[   14],
     | 30.00th=[   15], 40.00th=[   15], 50.00th=[   15], 60.00th=[   15],
     | 70.00th=[   16], 80.00th=[   16], 90.00th=[   17], 95.00th=[   27],
     | 99.00th=[   33], 99.50th=[   33], 99.90th=[  617], 99.95th=[  617],
     | 99.99th=[  651]
   bw (  KiB/s): min= 1003, max=32443, per=100.00%, avg=27646.17, stdev=5571.89, samples=366
   iops        : min=    1, max=   63, avg=53.52, stdev=10.97, samples=366
  lat (msec)   : 20=91.80%, 50=7.89%, 250=0.08%, 500=0.04%, 750=0.18%
  lat (msec)   : 1000=0.01%
  cpu          : usr=2.22%, sys=7.70%, ctx=15626, majf=0, minf=20
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,10000,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1
4kQD32read: (groupid=4, jobs=1): err= 0: pid=18539: Mon Nov 25 23:09:16 2024
  read: IOPS=2700, BW=10.5MiB/s (11.1MB/s)(5000MiB/474038msec)
    slat (usec): min=29, max=9036, avg=279.67, stdev=129.40
    clat (usec): min=270, max=112671, avg=11522.58, stdev=4061.02
     lat (usec): min=531, max=113014, avg=11802.26, stdev=4062.27
    clat percentiles (usec):
     |  1.00th=[ 1958],  5.00th=[ 4948], 10.00th=[ 8455], 20.00th=[10945],
     | 30.00th=[11076], 40.00th=[11207], 50.00th=[11338], 60.00th=[11469],
     | 70.00th=[11600], 80.00th=[11994], 90.00th=[13698], 95.00th=[17433],
     | 99.00th=[25035], 99.50th=[33162], 99.90th=[52167], 99.95th=[60031],
     | 99.99th=[74974]
   bw (  KiB/s): min= 9117, max=12328, per=100.00%, avg=10811.72, stdev=252.46, samples=943
   iops        : min= 2279, max= 3082, avg=2702.64, stdev=63.17, samples=943
  lat (usec)   : 500=0.01%, 750=0.01%, 1000=0.05%
  lat (msec)   : 2=1.00%, 4=2.61%, 10=8.70%, 20=85.06%, 50=2.46%
  lat (msec)   : 100=0.12%, 250=0.01%
  cpu          : usr=16.49%, sys=52.14%, ctx=1308604, majf=0, minf=52
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=1280000,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32
4kQD32write: (groupid=5, jobs=1): err= 0: pid=18543: Mon Nov 25 23:09:16 2024
  write: IOPS=1963, BW=7856KiB/s (8044kB/s)(5000MiB/651762msec); 0 zone resets
    slat (usec): min=50, max=200780, avg=379.95, stdev=1587.51
    clat (usec): min=307, max=1478.3k, avg=15817.26, stdev=20018.02
     lat (usec): min=536, max=1479.6k, avg=16197.21, stdev=20436.26
    clat percentiles (msec):
     |  1.00th=[   14],  5.00th=[   14], 10.00th=[   14], 20.00th=[   14],
     | 30.00th=[   14], 40.00th=[   14], 50.00th=[   14], 60.00th=[   14],
     | 70.00th=[   14], 80.00th=[   15], 90.00th=[   16], 95.00th=[   17],
     | 99.00th=[   69], 99.50th=[   74], 99.90th=[  211], 99.95th=[  472],
     | 99.99th=[  785]
   bw (  KiB/s): min=   55, max= 9184, per=100.00%, avg=7862.78, stdev=1429.49, samples=1298
   iops        : min=   13, max= 2296, avg=1965.43, stdev=357.39, samples=1298
  lat (usec)   : 500=0.01%, 750=0.01%, 1000=0.01%
  lat (msec)   : 2=0.01%, 4=0.01%, 10=0.04%, 20=96.79%, 50=1.13%
  lat (msec)   : 100=1.76%, 250=0.18%, 500=0.04%, 750=0.03%, 1000=0.01%
  lat (msec)   : 2000=0.01%
  cpu          : usr=17.75%, sys=40.13%, ctx=1352587, majf=0, minf=20
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,1280000,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=32.8MiB/s (34.4MB/s), 32.8MiB/s-32.8MiB/s (34.4MB/s-34.4MB/s), io=5000MiB (5243MB), run=152506-152506msec

Run status group 1 (all jobs):
  WRITE: bw=29.5MiB/s (30.9MB/s), 29.5MiB/s-29.5MiB/s (30.9MB/s-30.9MB/s), io=5000MiB (5243MB), run=169756-169756msec

Run status group 2 (all jobs):
   READ: bw=30.9MiB/s (32.4MB/s), 30.9MiB/s-30.9MiB/s (32.4MB/s-32.4MB/s), io=5000MiB (5243MB), run=161608-161608msec

Run status group 3 (all jobs):
  WRITE: bw=26.7MiB/s (28.0MB/s), 26.7MiB/s-26.7MiB/s (28.0MB/s-28.0MB/s), io=5000MiB (5243MB), run=187404-187404msec

Run status group 4 (all jobs):
   READ: bw=10.5MiB/s (11.1MB/s), 10.5MiB/s-10.5MiB/s (11.1MB/s-11.1MB/s), io=5000MiB (5243MB), run=474038-474038msec

Run status group 5 (all jobs):
  WRITE: bw=7856KiB/s (8044kB/s), 7856KiB/s-7856KiB/s (8044kB/s-8044kB/s), io=5000MiB (5243MB), run=651762-651762msec

Disk stats (read/write):
  mmcblk0: ios=1297262/1300114, merge=2742/229, ticks=4599359/963290, in_queue=5563829, util=77.80%
```
