I recommend you to disable CPU freq scaling for gain max performance (max frequency):
```bash
echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

You can use script [irq_balance_manually.sh](https://github.com/FastVPSEestiOu/fastnetmon/blob/master/src/irq_balance_manually.sh) for irq balancing on heavy loaded networks.

Running tool without root permissions:
```bash
useradd fastnetmon
setcap cap_net_admin+eip fastnetmon
su fastnetmon
./fastnetmon eth0,eth1
```

Please keep in mind when run tool on OpenVZ because without root permissions tool can't get all VE ips and you should pass it explicitly.


Debugging flags.

DUMP_ALL_PACKETS will enable all packets dumping to console. It's very useful for testing tool on non standard platforms.

```bash
DUMP_ALL_PACKETS=yes ./fastnetmon eth3,eth4
```

Recommended configuration options for ixgbe Intel X540 driver:
```bash
cat /etc/modprobe.d/ixgbe.conf
options ixgbe IntMode=2,2 MQ=1,1 DCA=2,2 RSS=8,8 VMDQ=0,0 max_vfs=0,0 L2LBen=0,0 InterruptThrottleRate=1,1 FCoE=0,0 LRO=1,1 allow_unsupported_sfp=0,0
```

I got very big packet size (more than mtu) in attack log? This issue will be related with offload features of NIC. For INtel 82599 I recommend disable all offload:
```bash
ethtool -K eth0 gro off gso off tso off
```


How I can enable hardware filtration for Intel 82599 NIC? Install patched ixgbe driver from PF_RING distro and apply this patch to Makefile and recompile tool:
```bash
fastnetmon.o: fastnetmon.cpp
-       $(COMPILER) $(STATIC) $(DEFINES) $(HEADERS) -c fastnetmon.cpp -o fastnetmon.o $(BUILD_FLAGS)
+       $(COMPILER) $(STATIC) $(DEFINES) $(HEADERS) -c fastnetmon.cpp -o fastnetmon.o $(BUILD_FLAGS) -DHWFILTER_LOCKING
```

How I can compile FastNetMon without PF_RING support?
```bash
cmake .. -DDISABLE_PF_RING_SUPPORT=ON
```

If you saw intel_idle in perf top with red higlihting you can disable it with following kernel params (more details you can find Performance_Tuning_Guide_for_Mellanox_Network_Adapters.pdf):
```bash
intel_idle.max_cstate=0 processor.max_cstate=1
```

If you want build with clang:
```bash
cmake -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ ..
```

If tou want build tool with debug info:
```bash
cmake -DCMAKE_BUILD_TYPE=Debug  ..
```

Performance tuning:
- Do not use short prefixes (lesser then /24)
- Do not use extremely big prefixes (/8, /16) because memory consumption will be very big

How I can enable ZC support? Please install DNA/ZC dreivers, load they and add interface name with zc prefix in config file (i.e. zc:eth3)

How I can optimally use ZC mode? You should enable number of NIC queues as number of logical cores in load_driver.sh 

You can find more info and graphics [here](http://forum.nag.ru/forum/index.php?showtopic=89703)
