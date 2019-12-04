ДЗ 3
1. Уменьшен том под / до 8G
 
устанавливаем пакет:
sudo -i 	
yum install -y xfsdump

подготавливаем временный том для раздлеа /

pvcreate /dev/sdb
vgcreate vg_root /dev/sdb
lvcreate -n lv_root -l +100%FREE /dev/vg_root

создаем файловую систему и монтируем 

mkfs.xfs /dev/vg_root/lv_root
mount /dev/vg_root/lv_root /mnt

копируем все даннýе с / раздела в /mnt

xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt

переконфигурируем grub

for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
chroot /mnt/
grub2-mkconfig -o /boot/grub2/grub.cfg

Обновляем образ initrd

cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;s/.img//g"` --force; done

правим /boot/grub2/grub.cfg(нужно заменить rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root/lv_root)

Делаем ребут

reboot

Уменьшаем размер старой VG и возвращаем на него рут

удалаем старый 

lvremove /dev/VolGroup00/LogVol00

создаем новый меньшего размера

lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00

Создаем файловую системы, монтируем на /mnt и копируем данные с / на /mnt

mkfs.xfs /dev/VolGroup00/LogVol00

mount /dev/VolGroup00/LogVol00 /mnt

xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt

переконфигурируем grub

for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
chroot /mnt/
grub2-mkconfig -o /boot/grub2/grub.cfg
cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;s/.img//g"` --force; done

делаем reboot

reboot

Выделяем том под /var в зеркало

На свободных дисках создаем зеркало

pvcreate /dev/sdc /dev/sdd
vgcreate vg_var /dev/sdc /dev/sdd
lvcreate -L 950M -m1 -n lv_var vg_var

создаем файловую систему и монтируем 

mkfs.ext4 /dev/vg_var/lv_var
mount /dev/vg_var/lv_var /mnt

копируем данные на /mnt

cp -aR /var/* /mnt/

Сохраняем на всякий случай старый var
mkdir /tmp/oldvar && mv /var/* /tmp/oldvar

монтируем var в каталог /var
umount /mnt
mount /dev/vg_var/lv_var /var

Правим fstab для автоматического монтирования /var

echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab

делаем ребут

reboot

После ребута можно удалить временнный Volume Group

lvremove /dev/vg_root/lv_root
vgremove /dev/vg_root
pvremove /dev/sdb


Выделяем том под /home

lvcreate -n LogVol_Home -L 2G /dev/VolGroup00

форматируем 

mkfs.xfs /dev/VolGroup00/LogVol_Home

монтируем на /mnt

mount /dev/VolGroup00/LogVol_Home /mnt/

копируем данные с /home на /mnt
cp -aR /home/* /mnt/ 

удаляем данные из /home

rm -rf /home/*

монтируем новый том в /home
umount /mnt
mount /dev/VolGroup00/LogVol_Home /home/

Правим fstab для автоматического монтирования /home

echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab

Восстановление со снапшота 

Сгенерируем файлы в /home/

touch /home/file{1..20}

Снимем снапшот

lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home

удаляем часть файлов
rm -f /home/file{11..20}

восстановимся со снапшота 
umount /home
lvconvert --merge /dev/VolGroup00/home_snap
mount /home

После этого убеждаемся что все файлы на месте





