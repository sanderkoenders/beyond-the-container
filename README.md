# beyond-the-container

## Experiment 1
Create two cgroups in the same hierarchy that each get a certain weight of CPU usage. When the weight is the same for each cgroup cpu power will be equally devided. To demonstrate this we will have to put both processes on the same core on the CPU. This can be done via cpuset.

```bash
sudo cgcreate -g cpu,cpuset:A
sudo cgcreate -g cpu,cpuset:B

sudo cgset -r cpuset.cpus=0 A
sudo cgset -r cpuset.cpus=0 B

sudo cgexec -g cpuset,cpu:A dd if=/dev/zero of=/dev/null &
sudo cgexec -g cpuset,cpu:B dd if=/dev/zero of=/dev/null &
```

## Experiment 2
When you create cgroups inside a cgroup next to another cgroup you'll see cpo percentages devides like 25%, 25%, 50%.
```bash
# Create cgroup G and F
sudo cgcreate -g cpuset,cpu:G
sudo cgcreate -g cpuset,cpu:F

# Create cgroup A and B inside cgroup F
sudo cgcreate -g cpu:F/A
sudo cgcreate -g cpu:F/B

# Set equal weight for cgroup F and G
sudo cgset -r cpu.weight=100 F
sudo cgset -r cpu.weight=100 G

# Set cgroup F and G to the same CPU
sudo cgset -r cpuset.cpus=1 F
sudo cgset -r cpuset.cpus=1 G

# Execute our dd commands
sudo cgexec -g cpu:F/A dd if=/dev/zero of=/dev/null &
sudo cgexec -g cpu:F/B dd if=/dev/zero of=/dev/null &
sudo cgexec -g cpuset,cpu:G dd if=/dev/zero of=/dev/null &
```

## Experiment 3
Create a cgroup and restrict access from devices on the host.
```bash
# Mount devices subsystem
sudo mkdir /sys/fs/cgroup/devices.mount 
sudo mount -t cgroup -o devices cgroup_devices /sys/fs/cgroup/devices.mount

# Create cgroup that can control devices
sudo cgcreate -g devices:C

# Disallow all devices
sudo cgset -r devices.deny=a C

# Allow writing to /dev/null
sudo cgset -r devices.allow='c 1:3 mrw' C

# Allow reading from /dev/zero
sudo cgset -r devices.allow='c 1:5 mr' C
```


