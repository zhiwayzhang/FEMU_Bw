# FEMU with Bandwidth Report

# Notes

## FEMU SSD init

using `mount.sh` mount femu to kernel

```shell
sudo mkdir /tmp/test && sudo mkfs.ext4 /dev/nvme0n1 && sudo mount /dev/nvme0n1 /tmp/test
```

## Generate IO traffic for testing

### dd

using `traffic.sh` generate backgroud traffic in `nvme0n1`

```shell
while true
do
    sudo dd if=/dev/nvme0n1 of=/tmp/test/file1 bs=3024M count=1
    sudo dd if=/dev/nvme0n1 of=/tmp/test/file2 bs=1000M count=1
done
```

run `traffic.sh` in background

```bash
nohup sudo sh traffic.sh > traffic.log 2>&1 &
```

### fio

using fio to test the performance of FEMU bbssd

```
[global]
ioengine=libaio
direct=1
time_based=1
runtime=350s
rw=randrw
bs=1M
numjobs=8
iodepth=64
size=1000M
group_reporting

[job1]
directory=/tmp/test
```

## Execute Nvme admin command in user level

using nvme-cli get the NAND utilization of femu bbssd, this command will be processed by `bb_flip` in `hw/femu/bbssd/bb.c`, but it's output as QEMU log, can't be access by VM.

```shell
sudo nvme admin-passthru /dev/nvme0n1 --opcode=0xef --cdw10=8
```

using nvme-cli get the NVMe Admin Command Result

```shell
echo $((0x$(echo $(sudo nvme admin-passthru /dev/nvme0n1 --opcode=0x03 --cdw10=1 2>&1) | cut -d ":" -f 2)))
```

<details>
<summary>click to show the explain of the command above</summary>
nvme-cli execute command and output nvme result as stderr output

redirect stderr to stdout

```shell
sudo nvme admin-passthru /dev/nvme0n1 --opcode=0x03 --cdw10=1 2>&1
```

output as

```
NVMe command result:03442f48
```

split output by `:`, filter the utilization value (index is [2])

```shell
cut -d ":" -f 2
```

finally, convert the hexadecimal value to decimal

```shell
echo $((0x{hex string}))
# example: echo $((0x03442f48))
```

the nvme result is generate by the rules below:

```shell
floor(total_utilization * 10000)*10000 + floor(gc_utilization * 10000)
```

if the nvme result is `0x03442f48(54800200 in decimal)`, the utilization will be:

```shell
total_utilization = 0.5480
gc_utlization = 0.0200
```

this allow total_utilization takes `4-5` digits, and gc_utilization takes `4` digits.

</details>

# Acknowledgement

based on [[FAST'18] FEMU](https://github.com/vtess/FEMU)
