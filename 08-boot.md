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

    # sed -i 's/centos/OtusRoot/'  /etc/default/grub
    # sed -i 's/centos/OtusRoot/'  /boot/grub2/grub.cfg


mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

# здесь не закончил чтото пошло не так при сборке и не смог загрузиться 


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


