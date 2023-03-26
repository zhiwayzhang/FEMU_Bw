# IO Scheduling

# Testing

use `mount.sh` mount femu to kernel
```shell
sudo mkdir /tmp/test && sudo mkfs.ext4 /dev/nvme0n1 && sudo mount /dev/nvme0n1 /tmp/test
```

use `traffic.sh` generate backgroud traffic in `nvme0n1`

```shell
while true
do
    sudo dd if=/dev/nvme0n1 of=/tmp/test/file1 bs=3024M count=1
    sudo dd if=/dev/nvme0n1 of=/tmp/test/file2 bs=1000M count=1
done
```

run `traffic.sh`

```bash
nohup sudo sh traffic.sh > traffic.log 2>&1 &
```

using nvme-cli get the NAND utilization of femu bbssd
```bash
sudo nvme admin-passthru /dev/nvme0n1 --opcode=0xef --cdw10=8
```

# Acknowledgement
based on [[FAST'18] FEMU](https://github.com/vtess/FEMU)