Урок: otus_NFS   
Автор: Kamil Ibragimov   
Дата: 26.05.2025   

## Домашнее задание: работа с ZFS
Задание:   
1. Определить алгоритм с наилучшим сжатием:   
• Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);   
• Cоздать 4 файловых системы на каждой применить свой алгоритм сжатия;   
• Для сжатия использовать либо текстовый файл, либо группу файлов.   
2. Определить настройки пула и при помощи команды zfs import собрать pool ZFS.   
Командами zfs определить настройки:     
• Размер хранилища;   
• Тип pool;   
• Значение recordsize;   
• Какое сжатие используется;   
• Какая контрольная сумма используется.   
3. Работа со снапшотами:   
• Скопировать файл из удаленной директории;   
• Сосстановить файл локально. zfs receive;   
• Найти зашифрованное сообщение в файле secret_message.   

Результат:   
• Создал 4 файловые системы с разными алгоритмами сжатия и скопировал лог файлы. Результат см. на скриншоте 🖼️ ["gzip"](https://github.com/kamil1403/otus_ZFS/blob/main/screenshots/gzip_zfs.png)  
• Импортировал пул и посмотрел его настройки. Результат см. на скриншоте 🖼️ ["pool"](https://github.com/kamil1403/otus_ZFS/blob/main/screenshots/pool_zfs.png)  
• Создал снапшот и восстановил файлы из него. Результат см. на скриншоте 🖼️ ["snapshot"](https://github.com/kamil1403/otus_ZFS/blob/main/screenshots/snapshot_zfs.png) 


## 🧭 Оглавление 🖼️

- [📦 Алгоритмы сжатия](#gzip)
- [⚙️ Импорт и настройки пула](#pool)
- [📸 Работа со снапшотами](#snapshot)
- [💡 Различные команды из урока](#other)

---

<a id="gzip"></a>
## 📦 Алгоритмы сжатия

```bash
# Создает новый пул   
zpool create otus_pool /dev/sdb /dev/sdc   
# Создает четыре файловые системы   
zfs create otus_pool/gzip_test_zfs   
zfs create otus_pool/lz4_test_zfs   
zfs create otus_pool/lzjb_test_zfs   
zfs create otus_pool/zle_test_zfs   
# Применяет сжатие   
zfs set compression=gzip otus_pool/gzip_test_zfs   
zfs set compression=lz4 otus_pool/lz4_test_zfs   
zfs set compression=lzjb otus_pool/lzjb_test_zfs   
zfs set compression=zle otus_pool/zle_test_zfs   
# Копирует для теста файлы логов   
cp -r /var/log/* /otus_pool/gzip_test_zfs   
cp -r /var/log/* /otus_pool/lz4_test_zfs   
cp -r /var/log/* /otus_pool/lzjb_test_zfs   
cp -r /var/log/* /otus_pool/zle_test_zfs   
# Показывает степерь сжатия файлов в каждом каталоге   
zfs get compressratio otus_pool  
zfs get compressratio otus_pool/gzip_test_zfs   
zfs get compressratio otus_pool/lz4_test_zfs   
zfs get compressratio otus_pool/lzjb_test_zfs   
zfs get compressratio otus_pool/zle_test_zfs 
```

---

<a id="pool"></a>
## ⚙️ Импорт и настройки пула

```bash|
# Показывает пулы, доступные для импорта  
zpool import
# Подключает пул
zpool import otus_pool
# Размер пула
zpool list otus_pool
# Тип пула
zpool status otus_pool
# Показывает степерь сжатия файлов в каждом каталоге   
zfs get compressratio otus_pool  
zfs get compressratio otus_pool/gzip_test_zfs   
zfs get compressratio otus_pool/lz4_test_zfs   
zfs get compressratio otus_pool/lzjb_test_zfs   
zfs get compressratio otus_pool/zle_test_zfs 
#Показывает контрольные суммы пула
zfs get checksum otus_pool
# Показыает все настройки пула
zfs get all otus_pool 
```

---

<a id="snapshot"></a>
## 📸 Работа со снапшотами

```bash
#Создает снапшот
zfs snapshot otus_pool/gzip_test_zfs@backup1
# Экспортирует снапшот в файл
zfs send otus_pool/gzip_test_zfs@backup1 > /tmp/backup1.zfs
# Удаляет все файлы из каталога
rm -rf /otus_pool/gzip_test_zfs/*
#Импортирует снапшот
zfs receive otus_pool/restored_backup < /tmp/backup1.zfs
zfs receive -F otus_pool/gzip_test_zfs < /tmp/backup1.zfs
# Показывает существующие снапшоты
zfs list -t snapshot
```

---

<a id="other"></a>
## 💡 Различные команды из урока

```bash
# Установка NFS-сервера
sudo apt update
sudo apt install nfs-kernel-server
# Настройка файла exports на сервере (/etc/exports) 
/mnt/raid01 *(rw,root_squash)
# Применяет экспорт
sudo exportfs -ra   
# Проверяет, что каталог расшаривается 
sudo exportfs -v 
# Монтирует каталог на клиенте
mount 192.168.1.99:/mnt/raid01 /mnt/ 
# или
mount -o vers=3 192.168.1.99:/mnt/raid01  /mnt
# Размонирует каталог
umount /mnt
# Показывает установленную версию NFS
dpkg -l | grep -i nfs 
# Показать экспортируемые каталоги
showmount -e 192.168.1.99
# Показыает смонитрованные каталоги
mount 
mount | grep 192.168.1.99
# Показыает протоколы
rpcinfo | grep nfs
# Редактирует файл nfs-kernel-service (/etc/default/)
nano /etc/default/nfs-kernel-service
# 


zfs create tmp_pool/zfs02   
zfs create tmp_pool/zfs03   
# Показывает все файловые системы ZFS с их размером, использованием и статусом   
zfs list   
# Показывает статус дедупликации (удаление дубликатов блоков данных)   
zfs get dedup   
# Включает дедупликацию на файловой системе zfs02   
zfs set dedup=on tmp_pool/zfs02   
# Показывает все параметры ZFS с выводом ошибок в стандартный поток   
zfs get all 2>&1 | less   
# Создает директории для теста   
mkdir /tmp_pool/zfs02/test01   
mkdir /tmp_pool/zfs02/test02   
# Копирует логи для теста   
cp -r /var/log/* tmp_pool/zfs01   
cp -r /var/log/* tmp_pool/zfs02   
# Показывает текущие параметры сжатия   
zfs get compression   
zfs get compressratio   
# Включает/выключает сжатие на zfs03   
zfs set compression=on tmp_pool/zfs03   
zfs set compression=off tmp_pool/zfs03   
# Удаляет файловую систему zfs03 и её содержимое (если примонтировано)   
rm -rf /tmp_pool/zfs03*   
# Удаляет файловую систему zfs01   
zfs destroy tmp_pool/zfs01   
# Удаляет пул
sudo zfs destroy -r tmp_pool
sudo zpool -f destroy tmp_pool
# Показывает установленные квоты на файловые системы   
zfs get quota   
# Ограничивает объём файловой системы zfs01 до 10 мегабайт
zfs set quota=10M tmp_pool/zfs01
# Отключает пул
zpool export otus_pool
```

---


