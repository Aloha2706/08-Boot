# Смена пароля . 

## Вход разными способами.
### Способ 1. init=/bin/sh
Грузим машину. В загрузочном меню жмем "e" В конце строки, начинающейся с linux16  init=/bin/sh. Нажимаем сtrl-x для загрузки в систему. 

Перемонтируем файловую систему в Read-Write режим.   

    #mount -o remount,rw /

### Способ 2. rd.break

В конце строки, начинающейся с linux16, добавляем rd.break
Это инициирует прерывание загрузки. 
Попадаем в emergency mode. 

    # mount -o remount,rw /sysroot
    # chroot /sysroot
    # passwd root
    # touch /.autorelabel

Перезагружаемся , входим с новым паролем. 

### Способ 3. rw init=/sysroot/bin/sh

В строке, начинающейся с linux16, заменяем ro на rw init=/sysroot/bin/sh
и собственно заходим в шел уже в режиме записи, и можем менять пароль. 


## Переименование VG LVM

[root@localhost ~]# vgs
  VG     #PV #LV #SN Attr   VSize   VFree
  centos   1   2   0 wz--n- <19.00g    0

[root@localhost ~]# vgrename centos OtusRoot
  Volume group "centos" successfully renamed to "OtusRoot"

[root@localhost ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <19.00g    0


Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. 
заменяем "centos" "OtusRoot"
Пересоздаем initramfs 

    mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

Перезагружаемся и если все сделали правильно. то загрузимся без ошибок. 


### Добавляем модуль в initrd

создаем папку

    mkdir /usr/lib/dracut/modules.d/01test

добавляем в нее два скрипта 

module-setup.sh

    #!/bin/bash

    check() {
        return 0
    }

    depends() {
        return 0
    }

    install() {
        inst_hook cleanup 00 "${moddir}/test.sh"
    }

test.sh

    #!/bin/bash

    exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
    cat <<'msgend'
    Hello! You are in dracut module!
    ___________________
    < I'm dracut module >
    -------------------
    \
        \
            .--.
        |o_o |
        |:_/ |
        //   \ \
        (|     | )
        /'\_   _/`\
        \___)=(___/
    msgend
    sleep 10
    echo " continuing...."

Пересобираем образ initrd предварительно скопировав старый образ куда нибудь

mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
или  dracut -f -v 

Проверяем добавился ли скрипт 

    #lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
    test

удаляем опции из grub.cfg
    
    sed -i 's/rhgb quiet//'  /boot/grub2/grub.cfg

При перезагрузке пингвина увидеть сложно, поэтому читаем 

     less /var/log/messages

    Oct 25 12:20:08 localhost dracut-pre-pivot: Hello! You are in dracut module!
    Oct 25 12:20:08 localhost dracut-pre-pivot: ___________________
    Oct 25 12:20:08 localhost dracut-pre-pivot: < I'm dracut module >
    Oct 25 12:20:08 localhost dracut-pre-pivot: -------------------
    Oct 25 12:20:08 localhost dracut-pre-pivot: \
    Oct 25 12:20:08 localhost dracut-pre-pivot: \
    Oct 25 12:20:08 localhost dracut-pre-pivot: .--.
    Oct 25 12:20:08 localhost dracut-pre-pivot: |o_o |
    Oct 25 12:20:08 localhost dracut-pre-pivot: |:_/ |
    Oct 25 12:20:08 localhost dracut-pre-pivot: //   \ \
    Oct 25 12:20:08 localhost dracut-pre-pivot: (|     | )
    Oct 25 12:20:08 localhost dracut-pre-pivot: /'\_   _/`\
    Oct 25 12:20:08 localhost dracut-pre-pivot: \___)=(___/
    Oct 25 12:20:18 localhost dracut-pre-pivot: continuing....

