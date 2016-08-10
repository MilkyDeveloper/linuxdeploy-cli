# Linux Deploy CLI

Copyright (C) 2015-2016 Антон Скшидлевский, GPLv3

Приложение с интерфейсом для командной строки, предназначенное для автоматизации процесса установки, конфигурирования и запуска GNU/Linux дистрибутивов внутри контейнера chroot. Приложение может работать как в обычных десктопных Linux-дистрибутивах, так и на мобильных платформах, основанных на ядре Linux, при условии соблюдения необходимых зависимостей (все зависимости могут быть собраны статически). Приложения из Linux-дистрибутива запускаются в chroot окружении, работают параллельно с основной системой и сопоставимы с ней по скорости. Поскольку работа Linux Deploy базируется на системном вызове ядра Linux, то в роли "гостевых" систем могут выступать только дистрибутивы Linux.

Приложение может работать в двух режимах: с правами суперпользователя (chroot) и без них (proot). В обычном режиме доступны все поддерживаемые типы установки: установка в файл, на раздел диска (логический диск), в POSIX совместимую директорию и в оперативную память (tmpfs). В режиме fakeroot доступна установка только в директорию, а также появляется ряд ограничений:
* все пользователи внутри контейнера имеют полный доступ ко всей файловой системе контейнера, а владельцем всех файлов и каталогов является текущий пользователь;
* нет доступа к привилегированным операциям с системой, например, не работает ping, ulimit и т.п.;
* приложения могут работать только с номерами сетевых портов выше 1024;
* если приложение в своей работе использует системный вызов chroot, то его необходимо запускать через специальную утилиту fakechroot, например fakechroot /usr/sbin/sshd -p 2222.

Приложение поддерживает автоматическую установку (базовой системы) и начальную настройку дистрибутивов Debian, Ubuntu, Kali Linux, Arch Linux, Fedora, CentOS, Gentoo, openSUSE и Slackware. Установка Linux-дистрибутива осуществляется по сети с официальных зеркал в интернете. Также поддерживается импорт любой другой системы из заранее подготовленного rootfs-ахрива в формате tar.gz, tar.bz2 или tar.xz. Приложение позволяет подключаться к консоли установленной системы (контейнеру), а также запускать и останавливать приложения внутри контейнера (есть поддержка различных систем инициализации и собственных сценариев автозапуска). Каждый вариант установки сохраняется в отдельный конфигурационный файл, который отвечает за настройку каждого контейнера. При необходимости, контейнеры можно запускать параллельно. Можно экспортировать конфигурацию и сам контейнер как rootfs-архив для последующего развертывания этого контейнера без повторной установки и настройки.

Для расширения возможностей приложения реализована модульная архитектура, каждый модуль здесь назван компонентом. Компоненты пишутся на Bash-совместимом языке сценариев Ash, каждый компонент представляет собой директорию с двумя основными файлами deploy.conf и deploy.sh. Реализация компонента сводится к написанию обработчиков для следующих действий: установка, настройка, запуск, остановка и вызов справки. Компоненты могут зависеть от других компонентов, есть защита от циклических зависимостей. В компоненте можно указать совместимость с конкретными версиями дистрибутивов, чтобы ограничить область его применения.

Зависимости:
* Ядро [Linux](http://kernel.org)
* [BusyBox](https://github.com/meefik/busybox) или Bash и набор GNU утилит
* [PRoot](http://proot.me) для работы без прав суперпользователя
* [QEMU](http://qemu.org), пакет [qemu-user-static](https://packages.debian.org/stable/qemu-user-static) для Debian (для эмуляции ахритектуры)
* Модуль ядра binfmt_misc (для поддержки режима эмуляции архитектуры без PRoot)

Использование:
```
USAGE:
   linuxdeploy [OPTIONS] COMMAND ...

OPTIONS:
   -p NAME - configuration profile
   -d - enable debug mode
   -t - enable trace mode

COMMANDS:
   config [...] [PARAMETERS] [NAME ...] - управление конфигурациями
      - без параметров выводит список конфигураций
      -r - удалить текущую конфигурацию
      -i FILE - импортировать конфигурацию
      -x - дамп текущей конфигурации
      -l - список с зависимостями для подключенных или указанных компонентов
      -a - список всех компонентов без учета совместимости
   deploy [-i|-c] [-n NAME] [NAME ...] - установка дистрибутива и подключенных компонентов
      -i - только установить, без конфигурирования
      -с - только конфигурировать, без установки
      -n NAME - пропустить установку указанного компонента
   import <FILE|URL> - импортировать rootfs-архив (tgz, tbz2 или txz) в текущий контейнер
   export <FILE> - экспортировать контейнер как rootfs-архив (tgz, tbz2 или txz)
   shell [-u USER] [APP] - смонтировать контейнер, если не смонтирован, и выполнить указанную команду внутри контейнера, по умолчанию /bin/bash
      -u USER - переключиться на указанного пользователя
   mount - смонтировать контейнер
   umount - размонтировать контейнер
   start [-m] [NAME ...] - запустить все подключенные компоненты или только указанные
      -m - смотрировать контейнер
   stop [-u] [NAME ...] - остановить все подключенные компоненты или только указанные
      -u - размонтировать контейнер
   sync <URL> - синхронизировать рабочее окружение с сервером
   status - отобразить состояние контейнера и компонетнов
   help [NAME ...] - вызвать справку
```

Справка по параметрам основных компонетнов:
```
PARAMETERS: 
   --method=chroot|proot
     Метод контейнеризации.

   --distrib=debian|ubuntu|kalilinux|fedora|centos|archlinux|gentoo|opensuse|slackware
     Кодовое имя дистрибутива, который будет установлен.

   --arch=NAME
     Архитектура сборки дистрибутива, например i386 для debian. См. информацию по конкретному дистрибутиву.

   --suite=NAME
     Версия дистрибутива, например wheezy для debian. См. информацию по конкретному дистрибутиву.

   --source-path=PATH
     Источник установки дистрибутива, можно указать адрес репозитория или путь к rootfs-ахриву.

   --target-path=PATH
     Путь установки, зависит от типа развертывания.

   --target-type=file|partition|directory|ram
     Вариант развертывания контейнера.

   --disk-size=SIZE
     Размер файла образа, когда выбран тип развертывания "file". Ноль означает автоматический выбор размера образа.

   --fs-type=ext2|ext3|ext4|auto
     Файловая система, которая будет создана внутри образа или на разделе.

   --emulator=PATH
     Указать какой использовать эмулятор, по умолчанию QEMU.

   --dns=IP|auto
     IP-адрес DNS сервера, можно указать несколько адресов через пробел.

   --mounts=SOURCE:TARGET ...
     Подключение ресурсов к контейнеру.

   --user-name=USER
     Имя пользователя, который будет создан после установки дистрибутива.

   --user-password=PASSWORD
     Пароль пользователя будет назначен указанному пользователю.

   --privileged-users=USERS
     Список пользователей через пробел, которых добавить в группы Android.

   --locale=LOCALE
     Локализация дистрибутива, например ru_RU.UTF-8.
```
