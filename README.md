Урок: otus_NFS   
Автор: Kamil Ibragimov   
Дата: 26.05.2025   

## Домашнее задание: работа с NFS
Цель:   
• Научиться самостоятельно разворачивать сервис NFS и подключать к нему клиентов.   

Задание:   
• Запустить две виртуальные машины (сервер NFS и клиента);   
• На сервере NFS должна быть подготовлена и экспортирована директория;   
• В экспортированной директории должна быть поддиректория с именем upload и правами на запись в неё;   
• Экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);  
• Монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3;  
• Cоздать bash-скрипт nfss_script.sh — для конфигурирования сервера, в которых описать bash-командами ранее выполненные шаги;   
• Cоздать bash-скрипт nfsc_script.sh — для конфигурирования клиента, в которых описать bash-командами ранее выполненные шаги.   

Результат:   
• Запустил две виртуальные машины (Server 192.168.1.99 и Client 192.168.1.100), настроил между ними связь через NFS. Шаги описал в [📦 Server](#nfs_ser) и [📦 Client](#nfs_cl)    
• Создал скрипт для конфигурирования NFS на сервере. Результат см. на скриншоте 🖼️ ["serv1"](https://github.com/kamil1403/otus_NFS/blob/main/screenshots/Server_NFS_bash_1.png) и 🖼️ ["serv2"](https://github.com/kamil1403/otus_NFS/blob/main/screenshots/Server_NFS_bash_2.png)   
• Создал скрипт для конфигурирования NFS на клиенте. Результат см. на скриншоте 🖼️ ["client1"](https://github.com/kamil1403/otus_NFS/blob/main/screenshots/Clietn_NFS_bash_1.png) и 🖼️ ["client2"](https://github.com/kamil1403/otus_NFS/blob/main/screenshots/Clietn_NFS_bash_2.png)  


## 🧭 Оглавление 🖼️

- [📦 Настройка NFS на сервере](#nfs_ser)
- [⚙️ Настройка NFS на клиенте](#nfs_cl)
- [✍🏻 Скрипт автоматической настройки NFS на сервере](#bash_ser)
- [✍🏻 Скрипт автоматической настройки NFS на клиенте](#bash_cl)
- [💡 Различные команды из урока](#other)

---

<a id="nfs_ser"></a>
## 📦 Настройка NFS на сервере (IP 192.168.1.99)

```bash
sudo apt update
sudo apt install nfs-kernel-server
mkdir -p /srv/share/upload
chown -R nobody:nogroup /srv/share
chmod 0777 /srv/share/upload 
nano /etc/exports
/srv/share 192.168.1.100/32(rw,sync,root_squash)
exportfs -ra 
sudo exportfs -s
cd /srv/share/upload
touch check_file_server
```

---

<a id="nfs_cl"></a>
## ⚙️ Настройка NFS на клиенте (IP 192.168.1.100)

```bash|
sudo apt install nfs-common
echo "192.168.1.99:/srv/share/ /mnt nfs vers=3,defaults 0 0" >> /etc/fstab
systemctl daemon-reload 
mount | grep mnt 
sudo reboot
showmount -a 192.168.1.99
```

---

<a id="bash_ser"></a>
## ✍🏻 Скрипт автоматической настройки NFS на сервере

```bash
#!/bin/bash

# Устанавливает пакет
sudo apt install -y nfs-kernel-server
# Создает каталог
mkdir -p /srv/share/upload
# Меняет владельца и группу для папки
chown -R nobody:nogroup /srv/share
# Открывает полные права
chmod 0777 /srv/share/upload
# Добавляет запись в файл для автоматического монтирования
echo "/srv/share 192.168.1.100/32(rw,sync,root_squash)" >> /etc/exports
# Применяет экспорт
exportfs -ra
# Проверяет, что каталог расшаривается 
sudo exportfs -s
# Переходит в каталог
cd /srv/share/upload
# Создает пустой файл в папке
touch check_file_server
```

---

<a id="bash_cl"></a>
## ✍🏻 Скрипт автоматической настройки NFS на клиенте

```bash
#!/bin/bash

# Устанавливает пакет
sudo apt install -y nfs-common
# Добавляет запись в файл для автоматического монтирования
echo "192.168.1.99:/srv/share/ /mnt nfs vers=3,defaults 0 0" >> /etc/fstab 
# Перезагружает конфигурацию
systemctl daemon-reload
# Монтирует все файловые системы
sudo mount -a
# Добавляет паузу в пять секунд
sleep 5
# Выводит список смонтированных ФС
mount | grep mnt
# Переходит в папку
cd /mnt/upload
# Создает пустой файл в папке
touch check_file_client
```

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
# Редактирует файл nfs-kernel-server (/etc/default/)
nano /etc/default/nfs-kernel-server
# Запрещает nfs v.3 (не работает)
RPCMOUNTDARGS="--manage-gids"
RPCMOUNTDOPTS="--no-nfs-version 3"
# Показывает службы (интересует /usr/sbin/rpc.mountd)
ps -efl | grep rpc
# Ищет расположение rpc службы (\ комментирует символы)
grep -r "\/usr\/sbin\/rpc\.mountd" /lib/systemd
# Редактирием службу
nano /lib/systemd/system/nfs-mountd.service
# Прописываем --no-nfs-version 3 ("костыльный" вариант)
ExecStart=/usr/sbin/rpc.mountd --no-nfs-version 3
# Перезапускаем службу 
systemctl daemon-reload
systemctl restart nfs-server.service
```

---


