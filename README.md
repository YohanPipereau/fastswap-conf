# fastswap-conf

I am testing communication between two VMs:
1. `fastswap1` on first node
2. `fastswap2` on second node

# command line

## Loading fastswap

VM2
===
```
/etc/init.d/opensmd start
sleep 1
ip addr add 10.0.0.2/24 dev ib0
ip link set up dev ib0
cd fastswap/farmemserver
./rmserver 50000
```

VM1
===
```
ip addr add 10.0.0.1/24 dev ib0
ip link set up dev ib0
cd fastswap/drivers
sudo insmod fastswap_rdma.ko sport=50000 sip="10.0.0.2" cip="10.0.0.1" nq=48
```

## Loading a swap device

VM1
===
```
fallocate -l 5G /swapfile
mkswap /swapfile
swapon /swapfile
```

## Activating cgroups

VM1
====
```
mkdir /cgroup2
./setup/init_bench_cgroups.sh
source gen_protocol.sh
```

## Running benchmark

VM1
===
```
$ ./benchmark.py kmeans 0.5 --id 5
Setting kmeans5 memory limit to 50% (2424M) of max
echo $$ > /cgroup2/benchmarks/kmeans5/cgroup.procs && OMP_NUM_THREADS=1 exec taskset -c 0 /usr/bin/time -v python3 /home/yohan/cfm/kmeans/kmeans.py
Exception in thread Thread-1:
Traceback (most recent call last):
  File "/usr/lib/python3.5/threading.py", line 914, in _bootstrap_inner
    self.run()
  File "/usr/lib/python3.5/threading.py", line 862, in run
    self._target(*self._args, **self._kwargs)
  File "/home/yohan/cfm/lib/workloads.py", line 65, in __exec
    assert(self.popen.returncode == 0)
AssertionError





Python Wall Time: -1606693890.7093391

Traceback (most recent call last):
  File "./benchmark.py", line 62, in <module>
    main()
  File "./benchmark.py", line 58, in main
    run_benchmark(args)
  File "./benchmark.py", line 34, in run_benchmark
    print_output(workload, args)
  File "./benchmark.py", line 12, in print_output
    usr_bin_time = workload.get_usr_bin_time()
  File "/home/yohan/cfm/lib/workloads.py", line 95, in get_usr_bin_time
    return parser.parse(self.stderr.decode('utf-8'))
  File "/home/yohan/cfm/lib/utils.py", line 58, in parse
    values = {'User Time': self.get_user_time(string),
  File "/home/yohan/cfm/lib/utils.py", line 66, in get_user_time
    return float(regex.search(string).groups()[0])
AttributeError: 'NoneType' object has no attribute 'groups'
```

# Kernel infos
 
On VM1:
=======
```
$ uname -a
Linux fastswap1 4.11.0-fastswap #1 SMP Thu Nov 19 11:57:42 CET 2020 x86_64 x86_64 x86_64 GNU/Linux
```
```
$ cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.11.0-fastswap root=UUID=37484c83-c39d-499e-8ecd-fd6c60ab90e9 ro cgroup_no_v1=memory
```

On VM2:
=======
```
$ uname -a
Linux fastswap2 4.11.0-fastswap #1 SMP Thu Nov 19 11:57:42 CET 2020 x86_64 x86_64 x86_64 GNU/Linux
```
```
$ cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.11.0-fastswap root=UUID=af15b427-b147-4b55-95ee-41cc859ced6f ro
```

# output of ‘free'

VM1
===
```
$ free -h
              total        used        free      shared  buff/cache   available
Mem:            18G        556M         17G        8.7M        577M         17G
Swap:          5.0G        400K        5.0G
```

VM2
===
```
$ free -h
              total        used        free      shared  buff/cache   available
Mem:            37G         32G        4.0G        8.7M        604M        4.0G
Swap:            0B          0B          0B
```

# dmesg log

* `dmesg_vm1.txt`: log of VM1

There are no custom drivers loaded on fastswap2

# Hardware specs

* `libvirt_vm1.xml` : libvirt configuration of VM1
* `libvirt_vm2.xml`: libvirt configuration of VM2

Mellanox Card: `Mellanox Technologies MT27500 Family [ConnectX-3]`
