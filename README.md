ДЗ 2

1. Добавлен 5й диск
2. создан raid массив уровня raid1
3. создан скрипт makeraid1.sh для созданя рейда
4. Создан раздел GPT на RAID
5. Созданы 5 партиций
6. На каждой из партиций из п.5 создана файловая система. 
7. Партиции из п.5 смонтированы в директории /raid/part{1,2,3,4,5}


8. сломаем рейд. для этого искусственно “зафейлим” одно блочное
устройство /dev/sde командой  mdadm /dev/md0 --fail /dev/sde

9. удаляем сломанный диск из массива mdadm /dev/md0 --remove /dev/sde
10. вставляем "новый" диск командой mdadm /dev/md0 --add /dev/sde. Командой mdadm -D /dev/md0 можно посмотреть состояние рейда.


11. прописан собранный рейд в конф, чтобы рейд собирался при загрузке

12*. создан Vagrantfile, который сразу собирает систему с подключенным рейдом 
