```
vagrant@minikube:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              1.6G  1.1M  1.6G   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  7.1G   22G  25% /
tmpfs                              7.8G     0  7.8G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  124M  1.7G   7% /boot
/dev/sda1                          1.1G  6.4M  1.1G   1% /boot/efi
vagrant                            461G   59G  402G  13% /vagrant
tmpfs                              1.6G  4.0K  1.6G   1% /run/user/1000

vagrant@minikube:~$ lsblk 
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  59.8M  1 loop /snap/core20/2321
loop1                       7:1    0  77.4M  1 loop /snap/lxd/29353
loop2                       7:2    0  33.7M  1 loop /snap/snapd/21761
loop3                       7:3    0  68.9M  1 loop /snap/core22/2049
loop4                       7:4    0     4K  1 loop /snap/bare/5
loop5                       7:5    0  66.6M  1 loop /snap/cups/1102
loop6                       7:6    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop7                       7:7    0 182.6M  1 loop /snap/chromium/3205
loop8                       7:8    0 493.5M  1 loop /snap/gnome-42-2204/201
sda                         8:0    0   100G  0 disk 
├─sda1                      8:1    0     1G  0 part /boot/efi
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0  60.9G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0  30.5G  0 lvm  /

vagrant@minikube:~$ sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv

vagrant@minikube:~$ sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

vagrant@minikube:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              1.6G  1.1M  1.6G   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   60G  7.2G   50G  13% /
tmpfs                              7.8G     0  7.8G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  124M  1.7G   7% /boot
/dev/sda1                          1.1G  6.4M  1.1G   1% /boot/efi
vagrant                            461G   59G  402G  13% /vagrant
tmpfs                              1.6G  4.0K  1.6G   1% /run/user/1000

vagrant@minikube:~$ lsblk 
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  59.8M  1 loop /snap/core20/2321
loop1                       7:1    0  77.4M  1 loop /snap/lxd/29353
loop2                       7:2    0  33.7M  1 loop /snap/snapd/21761
loop3                       7:3    0  68.9M  1 loop /snap/core22/2049
loop4                       7:4    0     4K  1 loop /snap/bare/5
loop5                       7:5    0  66.6M  1 loop /snap/cups/1102
loop6                       7:6    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop7                       7:7    0 182.6M  1 loop /snap/chromium/3205
loop8                       7:8    0 493.5M  1 loop /snap/gnome-42-2204/201
loop9                       7:9    0  59.6M  1 loop /snap/core20/2603
loop10                      7:10   0  79.5M  1 loop /snap/lxd/31335
sda                         8:0    0   100G  0 disk 
├─sda1                      8:1    0     1G  0 part /boot/efi
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0  60.9G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0  60.9G  0 lvm  /
```
