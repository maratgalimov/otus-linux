# Домашнее задание 1 

## Дано:
```
Обновить ядро в базовой системе
Цель: Студент получит навыки работы с Git, Vagrant, Packer и публикацией готовых образов в Vagrant Cloud.
В материалах к занятию есть методичка, в которой описана процедура обновления ядра из репозитория.
По данной методичке требуется выполнить необходимые действия.
Полученный в ходе выполнения ДЗ Vagrantfile должен быть залит в ваш репозиторий.
Для проверки ДЗ необходимо прислать ссылку на него.
Для выполнения ДЗ со * и ** вам потребуется сборка ядра и модулей из исходников.
Критерии оценки: Основное ДЗ - в репозитории есть рабочий Vagrantfile с вашим образом.
ДЗ со звездочкой: Ядро собрано из исходников
ДЗ с **: В вашем образе нормально работают VirtualBox Shared Folders
```
## Решение:

1. Установлено ядро kernel-ml
```
sudo yum install -y http://www.elrepo.org/elrepo-release-7.7-1.el7.elrepo.noarch.rpm
sudo yum --enablerepo elrepo-kernel install kernel-ml -y
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-set-default 0
reboot
```
2. Скачаны исходники ядра содержащие модуль файловой системы vboxfs в данном случае 5.6.0-rc2  
(в текущей стабильной версии ядра 5.5.4 поддержка vboxsf убрана)  
```
https://git.kernel.org/torvalds/t/linux-5.6-rc2.tar.gz
```
3. Установлены необходимые для компиляции ядра пакеты
```
sudo yum install -y ncurses-devel make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2 rpm-build
```
4. Создан конфиг-файл загруженного ядра c помощью make localmodconfig, с помощью make menuconfig  
   в ядро включены драйверы:
```
CONFIG_VBOXGUEST=y
CONFIG_VBOXSF_FS=y
```
5. Ядро скомпилированно и установлено

```
make -j 3 rpm-pkg
sudo rmp -ivh kernel-5.6.0_rc2-1.x86_64.rpm
```
6. Удалены старые ядра и пакеты необходимые для компиляции
```
sudo yum remove kernel-ml kernel.x86_64
sudo yum remove ncurses-devel make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2 rpm-build
```
7. Обновлено меню загрузки
```
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-set-default 0
```
8. В Vagrantfile изменена строка синхронизации папок.
```
config.vm.synced_folder ".", "/vagrant", disabled: false, type: "virtualbox"
```
9. После перезагрузки в виртуальной машине доступеп каталог хостовой машины примонтированный в точку монтирования /vagrant 
```
[vagrant@kernel-update ~]$ mount |grep vagrant
vagrant on /vagrant type vboxsf (rw,relatime)
```

## Свой vagrant box

1. Устанавливаем публичный ключ vagrant.pub
```
cat vagrant.pub > /home/vagrant/.ssh/authorized_keys
```
2. Создаем свой vagrant box
```
vagrant package --base 'local-test_kernel-update_1582053796020_23949' --output '5.6.0_rc2_vboxfs.box'
```
3. После проверок vagrant box, публикуем в vagrant cloud
```
vagrant cloud auth login
vagrant cloud publish --release maratgalimov/centos-7-5vboxfs 1.0 virtualbox /home/mauzer/otus/local-test/5.6.0_rc2_vboxfs.box
```
## В результате:

Vagrantfile описывает виртуальную машину создаваемую из самостоятельно собранного и опубликованного vagrant box  
При загрузке виртуалная машина монтирует каталог хост-машины в котором распологается Vagrantfile, в точку монтирования /vagrant посредством vboxfs


