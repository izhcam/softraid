#Software RAID6 with 6 disks
Удаление суперблока с дисков (если был ранее):
mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}

Создание RAID6 из 6 дисков:
mdadm --create --verbose /dev/md0 -l 6 -n 6 /dev/sd{b,c,d,e,f,g}

Создание конфига:
mkdir /etc/mdadm
"DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/{print}'>>/etc/mdadm/mdadm.conf

Назначение диска sdb сломанным, удаление его и добавление обратно:
mdadm /dev/md0 --fail /dev/sdb
mdadm /dev/md0 --remove /dev/sdb
mdadm /dev/md0 --add /dev/sdb

Создание метки GPT:
parted -s /dev/md0 mklabel gpt

Создание 5 партиций:
parted /dev/md0 mkpart primary ext4 0% 20%
parted /dev/md0 mkpart primary ext4 20% 40%
parted /dev/md0 mkpart primary ext4 40% 60%
parted /dev/md0 mkpart primary ext4 60% 80%
parted /dev/md0 mkpart primary ext4 80% 100%

Создание файловых систем на пяти партициях:
for i in $(seq 1 5); do mkfs.ext4 /dev/md0p$i; done

Создание точек монтирования:
mkdir -p /raid/part{1,2,3,4,5}

Монтирование:
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

Остановка RAID:
mdadm --stop md0
