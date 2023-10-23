# Смена пароля . 

## Вход разными способами.
### Способ 1. init=/bin/sh
Грузим машину. В загрузочном меню жмем "e" В конце строки, начинающейся с linux16  init=/bin/sh. Нажимаем сtrl-x для загрузки в систему. 

Перемонтируем файловую систему в Read-Write режим.   

    #mount -o remount,rw /

### Способ 2. rd.break

В конце строки, начинающейся с linux16, добавляем rd.break
Попадаем в emergency mode.

    # mount -o remount,rw /sysroot
    # chroot /sysroot
    # passwd root
    # touch /.autorelabel
    
Перезагружаемся , входим с новым паролем. 

### Способ 3. rw init=/sysroot/bin/sh
