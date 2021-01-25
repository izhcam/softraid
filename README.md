<H3> Software RAID6 with 6 disks<H3>
  
Удаление суперблока с дисков (если был ранее):<br>
_mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}_

Создание RAID6 из 6 дисков:<br>
_mdadm --create --verbose /dev/md0 -l 6 -n 6 /dev/sd{b,c,d,e,f,g}_

Создание конфига:<br>
_mkdir /etc/mdadm_<br>
_"DEVICE partitions" > /etc/mdadm/mdadm.conf_<br>
_mdadm --detail --scan --verbose | awk '/ARRAY/{print}'>>/etc/mdadm/mdadm.conf_<br>

Назначение диска sdb сломанным, удаление его и добавление обратно:<br>
_mdadm /dev/md0 --fail /dev/sdb_<br>
_mdadm /dev/md0 --remove /dev/sdb_<br>
_mdadm /dev/md0 --add /dev/sdb_<br>

Создание метки GPT:<br>
_parted -s /dev/md0 mklabel gpt_

Создание 5 партиций:<br>
_parted /dev/md0 mkpart primary ext4 0% 20%_<br>
_parted /dev/md0 mkpart primary ext4 20% 40%_<br>
_parted /dev/md0 mkpart primary ext4 40% 60%_<br>
_parted /dev/md0 mkpart primary ext4 60% 80%_<br>
_parted /dev/md0 mkpart primary ext4 80% 100%_<br>

Создание файловых систем на пяти партициях:<br>
_for i in $(seq 1 5); do mkfs.ext4 /dev/md0p$i; done_<br>

Создание точек монтирования:<br>
_mkdir -p /raid/part{1,2,3,4,5}_<br>

Монтирование:<br>
_for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done_<br>

Остановка RAID:<br>
_mdadm --stop md0_
