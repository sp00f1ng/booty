# booty

booty - программа для создания загрузочных образов операционных систем.

- [booty](#booty)
    - [About](#about)
    - [Quick Start](#quick-start)
    - [Interface](#Interface)
        - [booty build](#booty-build)
        - [booty linux](#booty-linux)
        - [booty ramdisk](#booty-ramdisk)
        - [booty image](#booty-image)
        - [booty run](#booty-run)
        - [import / export](#import--export)
    - [Boot Options](#boot-options)
        - [booty.use-shmfs](#booty.use-shmfs)
        - [booty.use-overlayfs](#booty.use-overlayfs)
        - [booty.copy-to-ram](#booty.copy-to-ram)
        - [booty.search-rootfs](#booty.search-rootfs)
        - [booty.rootfs-changes](#booty.rootfs-changes)
        - [booty.size-of-rootfs](#booty.size-of-rootfs)
        - [booty.init](#booty.init)
    - [Proof of Concept](#proof-of-concept)
    - [Examples](#examples)
    - [Known Issues](#known-issues)
    - [To-Do List](#to-do-list)

## About

Название происходит от слов **boot** to an**y**. Приложение работает с любым дистрибутивом GNU/Linux. Благодаря строгому соблюдению POSIX shell синтаксиса, приложение легко переносимо на любую архитектуру и любое устройство, и работает везде, где установлен POSIX shell.

* Система загружается с любого накопителя, например с USB-накопителя, система может целиком скопировать себя в оперативную память, а внешний USB-накопитель может быть извлечён из компьютера.

* Система может быть загружена в чистое tmpfs, а может использовать Overlay FS, и во втором случае будет представлена вся история изменений, сделанных в процессе работы системы. В буквальном смысле, вы сможете контролировать каждый файл, созданный системой.

* Система загружается в оперативную память, работает в оперативной памяти, но при этом может сохранить все изменения обратно на диск или в файл. На том же USB-накопителе может быть создан обычный файл, отформатирован в любую файловую систему, поддерживаемую ядром и использован для хранения всех изменений.

booty проводит весь процесс создания загрузочного образа "от" и "до".

* booty собирает ядро для операционной системы: проверяет наличие последней версии ядра, скачивает исходный код, компилирует и устанавливает.

* booty собирает первичный RAM-диск (initrd или initramfs) с системой, беря за основу хост-систему, где выполняется сборка загрузочного образа.

* booty создаёт файловую систему, копируя указанные пользователем данные. Поддерживается множество файловых систем, одна из которых SquashFS, но так же есть возможность обойтись без файловых систем вовсе.

* booty создаёт универсальный загрузочный образ с системой, который вы можете записать на USB-накопитель, либо вы можете взять данные из образа для дальнейшей загрузки по сети (PXE).

В отличии от других подобного рода приложений, booty имеет интуитивно простой и понятный интерфейс: нужно только указать директорию с установленной системой, чтобы получить загрузочный образ с этой системой.

booty предлагает новую философию в дистрибутиво-строении, полностью ломая все старые привычки использования операционных систем.

* Для создания загрузочного образа с системой нужно выполнить всего одну команду.

* Система настраивается на локальном компьютере. По-желанию тестируется в виртуальной машине на предмет работоспособности. Далее создаётся образ с системой и с ним можно загрузиться в любом месте. Система загружается уже настроенной и сразу после загрузки начинает решать задачи.

* Система может быть развёрнута на любом количестве машин, ничто не мешает строить целые кластеры из однотипных систем, загружая один и тот-же предварительно настроенный образ.

* Система загружается в оперативную память, вам больше не нужны дисковые накопители. Оперативная память - это и скорость работы, и износоустойчивость, и отказоустойчивость.

* Система одноразовая. Система загружается и работает до момента перезагрузки. Парадокс, но вы можете смело сломать систему, не боясь при этом ничего сломать.

* Резервное копирование системы больше не требуется. Загружаемая система и есть резервная копия сама по себе.

* Для обновления системы, переустановки или восстановления достаточно просто перезагрузиться, чтобы вернуть систему в исходное состояние.

**Внимание:** Система работает ровно до момента перезагрузки, поэтому для хранения важных данных используйте облачные решения.

## Quick Start

Установка, сборка и тестирование загрузочного образа выполняется в три простых шага.

```sh
make install

booty build

qemu-system-x86_64 -enable-kvm -m 1G -cdrom BOOT-x86_64.ISO
```

Возьмите любой дистрибутив GNU/Linux на ваш выбор. В качестве примера я предлагаю использовать Gentoo, но вы можете использовать любой установленный дистрибутив.

Для установки Gentoo архив нужно просто распаковать.

```sh
mkdir ~/gentoo

wget -P ~ https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20210407T214504Z/stage3-amd64-20210407T214504Z.tar.xz

tar xpvf ~/stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner -C ~/gentoo
```

Gentoo установлена. Для создания загрузочного образа нужно выполнить одну очень простую команду.

```sh
booty build gentoo.iso ~/gentoo
```

Подождите немного, пока booty соберёт все данные и выполнит сборку загрузочного образа. Первый раз это может занять продолжительное время, в зависимости от производительности вашего компьютера. Первый раз будет собрано ядро Linux.

После выполнения booty, вы получите новый файл BOOT-*.ISO, где вместо * будет имя используемой архитектуры.

Для проверки работоспособности загрузочного образа используйте виртуальную машину.

```sh
qemu-system-x86_64 -enable-kvm -m 1G -cdrom gentoo.iso
```

## Interface

Вы можете использовать программный интерфейс (API) в ваших скриптах для создания загрузочных образов.

**Внимание:** официально поддерживается только команда booty build, которая гарантирует результат. Все остальные представленные ниже команды ничто иное как внутренний интерфейс (API), который может меняться (читай: ломаться) от версии к версии, но автор постарается не ломать интерфейс без необходимости.

### booty build

```sh
booty build
```

Команда выполняет весь цикл сборки загрузочного образа.

```sh
booty build ДИРЕКТОРИЯ ДИРЕКТОРИЯ ДИРЕКТОРИЯ ...
```

Вы можете указать одну или несколько директорий, которые будут использованы для создания загрузочного образа.

```sh
booty build ОБРАЗ ДИРЕКТОРИЯ ...
```
Вы можете указать название файла с образом в начале или в конце списка. Это не обязательный параметр.

Значение по-умолчанию: `BOOT-$(uname -m).ISO`

```sh
booty build ДИРЕКТОРИЯ -- ПАРАМЕТРЫ ЗАГРУЗКИ
```

Через два минуса `--` вы можете задать параметры загрузки, которые будут использованы загрузчиком.

По-умолчанию вам будет предложено стандартное меню загрузчика с небольшой задержкой и список из нескольких вариантов для загрузки с различными опциями, но если установить параметры загрузки, то тогда система будет загружена моментально с указанными параметрами.

![GRUB2](https://github.com/sp00f1ng/booty/blob/htdocs/grub2-menu.png?raw=true)

```sh
cat .config | booty build
```

Вы можете указать свою конфигурацию ядра передав её через стандартный поток ввода / вывода.

По-умолчанию собирается ядро с конфигурацией `defconfig` и `kvm_guest.config`, а так же рядом других полезных (на личный взгляд автора) опций.

```sh
booty build ~/gnulinux.iso ~/gnulinux-rootfs/ ~/rootfs-changes/ -- net.ifnames=0 quiet < ~/.config
```

Это пример команды с полным набором всех возможных опций.

### booty linux

```sh
booty linux
```

Команда загружает исходный код Linux и собирает ядро, предоставляя готовые к установке файлы.

Все дополнительные параметры не обязательны.

Опция `--kernel-name` задаёт имя ядра, это всегда linux. Зарезервированная опция.

Опция `--kernel-version` задаёт версию ядра, которую необходимо загрузить.

Опция `--kernel-release` задаёт уникальное имя, когда необходимо собрать различные конфигурации одной и той же версии ядра.

Опция `--config-file` задаёт конфигурационный файл, с которым необходимо собрать ядро.

Опция `--install-path` задаёт путь, куда будут установлены файлы ядра.

Опция `--force` принудительно загружает исходный код ядра и собирает его.

По-умолчанию загружается последняя версия ядра Linux и собирается с конфигурацией `defconfig` и `kvm_guest.config`.

По-умолчанию ядро устанавливается в директорию `$PWD/root-XXXXXXXXXX`.

Исходный код ядра всегда сохраняется в директории `$XDG_CACHE_HOME/.booty` или `$HOME/.cache/booty`.

Архив `linux-release#version.pkg.tar` с файлами ядра так же всегда сохраняется в вышеупомянутой директории.

Переменная release задаётся автоматически, это SHA1-сумма файла конфигурации. При сборке одной и той же версии ядра с разной конфигурацией вы получите два разных ядра.

При повторном запуске команды, если ядро было собрано ранее, оно будет установлено сразу.

Для принудительной загрузки исходного кода и сборки ядра используйте опцию `--force`, либо очистите кэш.

Таким образом, запустив `while true; do booty linux; done` ядро будет загружено и собрано только один раз, а повторный запуск команды будет устанавливать уже готовое ядро.

### booty ramdisk

```sh
booty ramdisk
```

Команда собирает initrd или initramfs образ используя данные хост-системы, на которой была запущена.

Все дополнительные параметры не обязательны.

Опция `--install-path` задаёт директорию для установки базового окружения.

Опция `--image` задаёт имя образа.

По-умолчанию базовое окружение устанавливается в директорию `$PWD/ramdisk-XXXXXXXXXX`, после чего создаётся загрузочный образ `$PWD/ramdisk-XXXXXXXXXX`.

### booty image

```sh
booty image
```

Команда подготавливает загрузочный образ.

Все дополнительные параметры не обязательны.

Опция `--install-path` задаёт директорию, в которой будет собираться образ.

Опция `--image` задаёт имя образа.

Опция `local_file=image_file` задаёт имя файла, который следует поместить на образ. `local_file` должен иметь абсолютный путь, а `image_file` должен иметь путь относительно корня образа. Например, `/lib/modules/3.11/bzImage=boot/vmlinuz` помещает указанный локальный файл `/lib/modules/3.11/bzImage` в загрузочный образ на место `boot/vmlinuz`.

По-умолчанию сборка образа происходит в директории `$PWD/image-XXXXXXXXXX`, после чего создаётся загрузочный образ `$PWD/BOOT-$(uname -m).ISO`

### booty run

```sh
booty run ФУНКЦИЯ
```

Данная команда служит для отладки приложения и предназначена только для разработчиков.

Команда запускает любую внутреннюю функцию приложения. `booty run linux_via_http_get_version`

### import / export

Для сохранения и восстановления образов используйте специальные команды `booty import` и `booty export`.

Например, вы установили дистрибутив в директорию используя deboostrap, pacstrap и так далее.

Чтобы эту директорию сохранить отдельным файлом, используйте:

```sh
cd linux-chroot/
booty export > ~/vanilla-system-state.img
```

Затем, чтобы из файла установить дистрибутив в директорию, используйте:

```sh
cd linux-chroot/
booty import < ~/vanilla-system-state.img
```

Это может быть удобно, когда вы имеете множество конфигураций систем, но при этом "ванильный" образ системы всегда где-то хранится.

На самом деле вы можете использовать для этого любой более удобный инструмент.

## Boot Options

booty в процессе загрузки использует дополнительные опции, благодаря которым есть ряд интересных возможностей.

### booty.use-shmfs

Опция `booty.use-shmfs` указывает, что все данные должны быть извлечены в одну tmpfs-директорию перед загрузкой.

К корневой раздел с системой будет подключён как `tmpfs /`.

**Осторожно:** Данный метод загрузки использует много оперативной памяти.

Если ваша система собрана в initramfs, то на короткий промежуток времени (на время распаковки) потребуется ещё столько же свободной оперативной памяти, сколько занимает сам initramfs. До тех пор, пока initramfs не будет целиком распакован в tmpfs корень и не переключится в него.

При работе в tmpfs следует быть осторожным с опцией `booty.size-of-rootfs` и не выделять слишком большой процент объёма оперативной памяти под корневой раздел. К примеру, из-за роботов, которые брутфорсят SSH, размер /var/log постоянно растёт, и рано или поздно весь объём оперативной памяти может быть занят. Для решения этой проблемы вы можете смонтировать /var/log в отдельный tmpfs раздел, используя опцию `booty.volume`.

### booty.use-overlayfs

Опция `booty.use-overlayfs` монтирует все имеющиеся разделы как слои, накладывая их друг на друга.

Использовать SquashFS для этого не обязательно. Всё, что было так или иначе добавлено в загрузку, будет смонтировано с использованием файловой системы Overlay FS: все прочие файловые системы, директории, блочные устройства...

**Интересный факт:** При использовании Overlay FS все изменения, которые происходят в системе, сохраняются в отдельной директории `/mnt/overlay_fs/rootfs-changes`. В буквальном смысле вы можете контролировать каждый новый файл в системе `find /mnt/overlay_fs/rootfs-changes`, либо же сохранять все изменения с удалённого хоста `scp -r REMOTE:/mnt/overlay_fs/rootfs-changes .`.

### booty.copy-to-ram

По-умолчанию, система загружается и работает с USB-накопителя. Опция `booty.copy-to-ram` копирует все данные с USB-накопителя в оперативную память, после чего устройство можно отключить от компьютера, а система продолжает загрузку уже из оперативной памяти.

### booty.search-rootfs

Опция `booty.search-rootfs` ищет заданный файл или директорию с системой на всех имеющихся блочных устройствах, и при нахождении загружается в неё.

### booty.rootfs-changes

По-умолчанию, при загрузке с использованием Overlay FS, все данные остаются в оперативной памяти и отображаются в директории `/mnt/overlay_fs/rootfs-changes`.

Опция `booty.rootfs-changes` позволяет задать блочное устройство или файл, куда следует сохранять все изменения. При этом блочное устройство или файл уже должны содержать в себе файловую систему, поддерживаемую ядром и доступную для записи.

Например, `booty.rootfs-changes=/dev/sda1` будет использовать `/dev/sda1` для хранения всех данных.

### booty.size-of-rootfs

Опция `booty.size-of-rootfs` задаёт размер для корневого раздела в tmpfs, куда будет загружена система.

По-умолчанию размер составляет 50% от объёма оперативной памяти.

Если во время загрузки система зависает, но вы уверены, что объёма оперативной памяти должно хватать в притык, попробуйте задать `booty.size-of-rootfs=80%`.

### booty.init

Опция `booty.init` задаёт программу инициализации во время загрузки.

По-умолчанию `booty.init=/sbin/init`.

## Proof of Concept

Скриншоты различных дистрибутивов, доказывающие, что booty просто работает.

Дистрибутивы были установлены самым обычным образом на внешний накопитель.

Корневая файловая система смонтирована в `/mnt`.

Загрузочный образ был создан запуском команды `booty build /mnt`.

Никаких специальных настроек над дистрибутивами произведено не было.

Вывод `mount | grep rootfs` сообщает, что система загружена с использованием Overlay FS.

Alpine Linux

![Alpine](https://github.com/sp00f1ng/booty/blob/htdocs/alpine-linux.png?raw=true)

Gentoo Linux

![Gentoo](https://github.com/sp00f1ng/booty/blob/htdocs/gentoo-linux.png?raw=true)

Arch Linux

**Внимание:** для создания загрузочного образа с Arch Linux необходимо использовать bootstrap, а не устанавливать систему с ISO-образа.

![Arch](https://github.com/sp00f1ng/booty/blob/htdocs/arch-linux.png?raw=true)

![Arch](https://github.com/sp00f1ng/booty/blob/htdocs/arch-linux-failed.png?raw=true)

Debian

![Debian](https://github.com/sp00f1ng/booty/blob/htdocs/debian-linux.png?raw=true)

CRUX

![CRUX](https://github.com/sp00f1ng/booty/blob/htdocs/crux-linux.png?raw=true)

## Examples

В разделе собраны все ответы на частозадаемые вопросы и случаи использования booty в примерах.

###### Собрать образы, которые отличаются между собой только рядом файлов: /etc/hostname, ~/.ssh

Установите систему любым удобным способом в директорию `linux-install/`.

Создайте несколько директорий:

В `host-master/` создайте файл `etc/hostname` с текстом `master`.

В `host-mistress/` создайте файл `etc/hostname` с текстом `mistress`.

В `host-slave/` создайте файл `etc/hostname` с текстом `slave`.

Теперь создайте образы:

`booty build master.iso linux-install/ host-master/`

`booty build mistress.iso linux-install/ host-mistress/`

`booty build slave.iso linux-install/ host-slave/`

Это одинаковые образы, которые отличаются между собой только содержимым host-директорий.

Вы храните дистрибутив в одной директории, а настройки в другой. Для обновления вы удаляете директорию с дистрибутивом и устанавливаете свежую версию. Вся конфигурация остаётся на месте.

## Known Issues

### init as symlink

Известный баг, когда booty не может загрузиться в систему, если `/sbin/init` это симлинк, указывающий например, на `/bin/busybox`, в то время как текущий `/` (корень) это всё ещё корень самого initramfs, а не системы, в которую booty собирается загрузиться.

`readlink`???

## To-Do List

* Переписать booty-init.in, избавиться от лапше-кода.

* Научить booty создавать обычные дисковые образы, готовые к установке путём обычного `dd`, т.е. нужно создавать пустой raw-файл, разбивать на разделы `fdisk`, форматировать в различные файловые системы на выбор `mke2fs`, `ntfs3g` и другие, и в конечном счёте устанавливать туда систему как на типичное блочное устройство. Файл должен быть готов к записи с использованием `dd`.

* Научить booty собирать не только ISO для x86/amd64 архитектур, но и другие образы для других систем включая ARM/ARM64, специфичные образы для одноплатников и т.д. Нужен Raspberry Pi для экспериментов.

* Вынести профили систем в отдельные файлы конфигурации, чтобы пользователи могли самостоятельно писать свои собственные профили для создания образов.

---

По всем вопросам, пожеланиям и предложениям пишите на форуме <a href="https://www.linux.org.ru/forum/">www.linux.org.ru/forum/</a> с пометкой <b>booty</b>.
