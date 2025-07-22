# Modern OS Development: Comprehensive Guide to Building an Operating System from Scratch
**Форк проекта [os-project](https://github.com/thedenisnikulin/os-project) с фундаментальной переработкой для современных реалий разработки системного ПО**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/github/actions/workflow/status/alexeev-prog/os-project-modern/ci.yml?label=continuous%20integration)](https://github.com/alexeev-prog/os-project-modern/actions)
![Project Status](https://img.shields.io/badge/status-active_development-orange)
[![NixOS Support](https://img.shields.io/badge/NixOS-supported-7d81ce)](https://nixos.org)

## 🔍 О проекте

**Modern OS Development** — это глубоко переработанный форк оригинального проекта [os-project](https://github.com/thedenisnikulin/os-project), созданный с целью предоставить русскоязычному сообществу наиболее полное и современное руководство по разработке операционных систем. В отличие от многих поверхностных туториалов, мы последовательно разбираем каждую компоненту ОС, начиная с низкоуровневых аспектов загрузки и заканчивая реализацией системных сервисов.

> **Текущий статус**: Проект находится в активной разработке. Для стабильной версии с базовой функциональностью рекомендуется использовать [оригинальный os-project](https://github.com/thedenisnikulin/os-project). Наша цель — создать наиболее полный русскоязычный ресурс по системному программированию, который будет включать как фундаментальные концепции, так и современные практики.

## 🧠 Философия проекта

Мы придерживаемся следующих принципов в разработке:

1. **Глубина вместо ширины** — Лучше детально разобрать ключевые механизмы ОС, чем поверхностно охватывать множество тем
2. **Актуальность инструментов** — Использование современных компиляторов (LLVM), протоколов (UEFI) и методологий
3. **Практическая ориентация** — Каждая теоретическая концепция сопровождается работающим кодом
4. **Воспроизводимость** — Nix-окружение гарантирует идентичные результаты сборки на любой системе
5. **Документирование для рунета** — Объяснение сложных концепций на русском языке без необходимости обращения к англоязычной документации

## ⚙️ Техническая архитектура

### Уровневая модель реализации

```
+-------------------------------------+
|          Пользовательские           |
|           приложения (apps)         |
+-------------------------------------+
|    Системные сервисы (filesystem,   |
|          networking, GUI)           |
+-------------------------------------+
|       Драйверы устройств (drivers)  |
+-------------------------------------+
|     Абстракция оборудования (hal)   |
+-------------------------------------+
|      Ядро ОС (scheduler, memory     |
|      management, system calls)      |
+-------------------------------------+
|        Архитектурно-зависимый       |
|           код (arch/x86_64)         |
+-------------------------------------+
|          Загрузчик (boot)           |
+-------------------------------------+
|             UEFI/BIOS               |
+-------------------------------------+
```

### Детализация компонентов

**UEFI Загрузчик**
Реализует современный протокол загрузки вместо устаревшего BIOS, поддерживает GPT-разделы и безопасную загрузку (Secure Boot). Включает:

- Инициализацию графического режима
- Загрузку ядра и initrd в защищённую память
- Парсинг ACPI-таблиц
- Переход в 64-битный режим

**Микроядро архитектуры**
Реализует базовые механизмы ОС с чётким разделением ответственности:

1. **Управление памятью**:
   - 4-уровневая страничная организация (PML4 → PDPT → PD → PT)
   - Блочные аллокаторы (buddy allocator)
   - Slab-аллокаторы для объектов ядра

2. **Многозадачность**:
   - Поддержка SMP (Symmetric Multiprocessing)
   - Вытесняющее планирование с квантованием времени
   - Приоритетные очереди

3. **Межпроцессное взаимодействие**:
   - Очереди сообщений
   - Разделяемая память
   - Сигналы

**Драйверная модель**
Унифицированный интерфейс для работы с оборудованием:

```c
struct device_driver {
    char *name;
    int (*init)(struct device *dev);
    int (*read)(struct device *dev, void *buf, size_t count);
    int (*write)(struct device *dev, const void *buf, size_t count);
    int (*ioctl)(struct device *dev, unsigned int cmd, void *arg);
    struct list_head list;
};
```

**Файловая система**
Многослойная архитектура:

```
Пользовательское пространство
       ↓
VFS (Virtual File System)
       ↓
Ext2/FAT32/ZFS реализации
       ↓
Блочные устройства (NVMe/SATA/VirtIO)
```

## 📦 Установка и настройка окружения

### Системные требования

- **Процессор**: x86_64 с поддержкой виртуализации (Intel VT-x/AMD-V)
- **Операционная система**: Любое современное Linux-распределение
- **Необходимые пакеты**:

```bash
# Для Debian/Ubuntu
sudo apt install build-essential nasm qemu-system-x86 ovmf \
     clang lld cmake ninja-build git mtools

# Для Fedora/CentOS
sudo dnf groupinstall "Development Tools"
sudo dnf install nasm qemu edk2-ovmf clang lld cmake ninja-build git mtools
```

### Настройка Nix-окружения (опционально, но рекомендуется)

```nix
# flake.nix
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  outputs = { self, nixpkgs }: {
    devShells.x86_64-linux.default = let
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
    in pkgs.mkShell {
      packages = [
        pkgs.qemu
        pkgs.llvmPackages_17.clang
        pkgs.lld
        pkgs.nasm
        pkgs.cmake
        pkgs.ninja
        pkgs.ovmf
        pkgs.mtools
        pkgs.gdb
      ];

      shellHook = ''
        echo "Среда разработки Modern OS готова"
        echo "Используемые версии:"
        clang --version | head -n 1
        lld --version | head -n 1
        nasm -v
      '';
    };
  };
}
```

### Сборка и запуск ОС

```bash
# Клонирование репозитория
git clone https://github.com/alexeev-prog/os-project-modern
cd os-project-modern

# Инициализация сборки (CMake)
mkdir build && cd build
cmake -DCMAKE_TOOLCHAIN_FILE=../cmake/x86_64-elf-toolchain.cmake ..
cmake --build . --target os-image -j$(nproc)

# Запуск в QEMU с UEFI
qemu-system-x86_64 \
  -machine q35,accel=kvm \
  -cpu host \
  -smp 4 \
  -m 4G \
  -bios /usr/share/ovmf/OVMF.fd \
  -drive file=./os-image,format=raw \
  -serial stdio \
  -netdev user,id=net0 \
  -device virtio-net-pci,netdev=net0
```

## 🔧 Ключевые технические концепции

### Переход в 64-битный режим

Процесс инициализации Long Mode включает несколько критических этапов:

1. **Настройка страничной организации памяти**
   Конфигурация PML4, PDPT, PD таблиц для переключения в 64-битный режим

2. **Активация PAE (Physical Address Extension)**
   Включение бита PAE в регистре CR4 для поддержки физических адресов >32 бит

```asm
; Пример настройки страничных таблиц
setup_paging:
    ; Создание PML4
    mov eax, pdp_table
    or eax, 0x03  ; Present + Writable
    mov [pml4_table], eax

    ; Создание PDPT
    mov eax, pd_table
    or eax, 0x03
    mov [pdp_table], eax

    ; Активация PAE
    mov eax, cr4
    or eax, 0x20
    mov cr4, eax

    ; Установка PML4
    mov eax, pml4_table
    mov cr3, eax

    ; Активация Long Mode
    mov ecx, 0xC0000080
    rdmsr
    or eax, 0x0100
    wrmsr

    ; Включение пейджинга
    mov eax, cr0
    or eax, 0x80000000
    mov cr0, eax
```

### Обработка прерываний в защищённом режиме

Современная архитектура обработки прерываний включает:

1. **Программируемый контроллер прерываний (APIC)**
   - Локальный APIC (LAPIC) на каждом ядре
   - I/O APIC для маршрутизации прерываний

2. **Таблица векторов прерываний (IDT)**
   - 256 дескрипторов прерываний
   - Шлюзы ловушек и прерываний

3. **Контекст переключения**
   Сохранение полного состояния процессора при обработке прерывания

```c
// Структура состояния процессора
struct cpu_state {
    uint64_t r15, r14, r13, r12, r11, r10, r9, r8;
    uint64_t rdi, rsi, rbp, rbx, rdx, rcx, rax;
    uint64_t int_num, err_code;
    uint64_t rip, cs, rflags, rsp, ss;
};

// Обработчик прерывания
__attribute__((interrupt))
void isr_handler(struct cpu_state *frame) {
    switch(frame->int_num) {
        case 0x0E: // Page Fault
            handle_page_fault(frame);
            break;
        case 0x20: // Timer Interrupt
            schedule();
            break;
        // ...
    }
}
```

### Виртуальная файловая система (VFS)

Архитектура VFS обеспечивает абстракцию над различными ФС:

```c
struct vfs_node {
    char name[256];
    uint32_t flags;
    uint32_t inode;
    uint64_t size;

    struct file_operations *fops;
    struct inode_operations *iops;
    struct super_operations *sops;

    void *private_data;
};

struct file_operations {
    ssize_t (*read)(struct vfs_node *, char *, size_t, off_t *);
    ssize_t (*write)(struct vfs_node *, const char *, size_t, off_t *);
    int (*open)(struct vfs_node *);
    int (*close)(struct vfs_node *);
};

// Регистрация файловой системы
int register_filesystem(const char *name, struct file_system_type *fs) {
    // Добавление в глобальный список ФС
}
```

## 🗺️ Детальная дорожная карта разработки

### Этап 1: Базовое ядро (Q3-Q4 2025)

1. **Современная система сборки**
   - Полная интеграция CMake/Ninja
   - Кросс-компиляция с использованием LLVM/Clang
   - Автоматизированная генерация зависимостей
   - Поддержка LTO (Link Time Optimization)

2. **Управление памятью**
   - Реализация 4-уровневой страничной организации
   - Аллокаторы физической памяти (bitmap, zone-based)
   - Виртуальные аллокаторы (slab, kmalloc)
   - Поддержка DMA-памяти

3. **Многозадачность**
   - Контекст переключения с сохранением FPU/SSE
   - Планировщик с поддержкой приоритетов
   - Механизмы синхронизации (мьютексы, семафоры)
   - Поддержка SMP через ACPI MADT

4. **Драйверная инфраструктура**
   - Унифицированная модель драйверов
   - Подсистема обнаружения устройств (PCI/PCIe scanning)
   - Драйверы: UART, PS/2, ATA, VGA

### Этап 2: Системные сервисы (Q1-Q2 2026)

1. **Файловые системы**
   - Виртуальная файловая система (VFS)
   - Реализация FAT32 с полной поддержкой чтения/записи
   - Экспериментальная поддержка ext2
   - Кэширование блоков

2. **Сетевая подсистема**
   - Драйвер VirtIO-net
   - Реализация TCP/IP стека на базе lwIP
   - Поддержка ARP, ICMP, UDP
   - Базовый сетевой стек (сокеты)

3. **Пользовательский интерфейс**
   - Текстовый режим с улучшенным терминалом
   - Режим фреймбуфера с поддержкой VESA
   - Базовый оконный менеджер
   - Подсистема ввода (клавиатура+мышь)

### Этап 3: Расширенные возможности (Q3-Q4 2026)

1. **Безопасность**
   - Реализация ASLR (Address Space Layout Randomization)
   - Поддержка SMEP/SMAP (Supervisor Mode Execution Prevention)
   - Изоляция драйверов в пользовательском пространстве

2. **Оптимизации**
   - Компиляция ядра с PGO (Profile Guided Optimization)
   - Поддержка KASLR (Kernel Address Space Layout Randomization)
   - Интеграция zstd для сжатия initrd

3. **Поддержка оборудования**
   - USB 3.x (xhci)
   - NVMe драйвер
   - Поддержка UEFI Runtime Services

## 📚 Образовательные ресурсы и ссылки

### Основная литература

1. **Intel® 64 and IA-32 Architectures Software Developer Manuals**
   Обязательное чтение для понимания архитектуры x86_64
   [Volume 1-3](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)

2. **UEFI Specification**
   Полное описание протокола UEFI и его реализации
   [Version 2.10](https://uefi.org/specifications)

3. **Operating Systems: Three Easy Pieces**
   Фундаментальные концепции ОС в доступной форме
   [Онлайн-версия](https://pages.cs.wisc.edu/~remzi/OSTEP/)

### Дополнительные материалы

1. **Современные практики разработки ОС**
   [OSDev Wiki](https://wiki.osdev.org) - Энциклопедия разработчика ОС
   [Linux Inside (рус.)](https://github.com/proninyaroslav/linux-insides-ru) - Детальное описание ядра Linux

2. **Оптимизация низкоуровневого кода**
   [Agner Fog's Optimization Manuals](https://www.agner.org/optimize/)
   [LLVM Performance Tips](https://llvm.org/docs/PerformanceTips.html)

3. **Отладка системного ПО**
   [QEMU+GDB: Практическое руководство](https://nixos.wiki/wiki/Debugging)
   [UEFI Debugging Techniques](https://tianocore-docs.github.io/edk2-Debugging/)

## 🤝 Участие в разработке

Мы приглашаем к сотрудничеству:
- Системных программистов для разработки ядра
- Специалистов по безопасности для реализации защитных механизмов
- Технических писателей для документации
- Тестировщиков для проверки на различном оборудовании

**Как внести вклад:**
```bash
# 1. Клонирование репозитория
git clone https://github.com/alexeev-prog/os-project-modern
cd os-project-modern

# 2. Настройка окружения (для разработчиков)
mkdir build-debug && cd build-debug
cmake -DCMAKE_BUILD_TYPE=DEBUG -DENABLE_SANITIZERS=ON ..

# 3. Сборка с включенными инструментами отладки
cmake --build . --target os-image

# 4. Запуск тестов
ctest -V

# 5. Создание pull request с описанием изменений
```

**Основные направления для контрибьюции:**
- Реализация драйверов для современного оборудования
- Портирование на архитектуру ARMv8 (AArch64)
- Оптимизация критических участков кода
- Перевод документации на английский язык
- Создание интерактивных учебных материалов

## 📜 Лицензионная информация

Проект распространяется под двойной лицензией:
- **Исходный код**: MIT License
- **Документация**: Creative Commons Attribution-ShareAlike 4.0

> *"Наша миссия — создать открытый образовательный ресурс, который позволит русскоязычным разработчикам глубоко понять внутреннее устройство операционных систем без необходимости изучения англоязычной документации. Каждая строка кода и каждая страница документации — это вклад в развитие системного программирования в рунете."*
> — Команда разработчиков Modern OS Project

**Технические дискуссии**: [Telegram-чат проекта](https://t.me/modern_os_dev)
**Отчёты об ошибках**: [GitHub Issues](https://github.com/alexeev-prog/os-project-modern/issues)
**План разработки**: [GitHub Projects](https://github.com/alexeev-prog/os-project-modern/projects)

## Ссылки
Ссылки на полезный материал которым я пользовался в качестве теории.

> На русском языке:
- Серия статей о ядре Linux и его внутреннем устройстве: https://github.com/proninyaroslav/linux-insides-ru
- Статья "Давай напишем ядро!": https://xakep.ru/2018/06/18/lets-write-a-kernel/
- nasm docs: https://www.opennet.ru/docs/RUS/nasm/nasm_ru3.html
> На английском языке:
- Небольшая книга по разработке собственной ОС (70 страниц): https://www.cs.bham.ac.uk/~exr/lectures/opsys/10_11/lectures/os-dev.pdf
- Общее введене в разработку операционных систем: https://wiki.osdev.org/Getting_Started
- Туториал по разработке ядра операционной системы для 32-bit x86 архитектуры. Первые шаги в создании собсвтенной ОС: https://wiki.osdev.org/Bare_Bones
- Продолжение предыдущего туториала: https://wiki.osdev.org/Meaty_Skeleton
- Про загрузку ОС (booting): https://wiki.osdev.org/Boot_Sequence
- Список туториалов по написанию ядра и модулей к ОС: https://wiki.osdev.org/Tutorials
- Внушительных размеров гайд по разработке ОС с нуля: http://www.brokenthorn.com/Resources/OSDevIndex.html
- Книга, описывающая ОС xv6 (не особо вникал, но должно быть что-то годное): https://github.com/mit-pdos/xv6-riscv-book, сама ОС: https://github.com/mit-pdos/xv6-public
- "Небольшая книга о разработке операционных систем" https://littleosbook.github.io/
- Операционная система от 0 до 1 (книга): https://github.com/tuhdo/os01
- ОС, написанная как пример для предыдущей книги: https://github.com/tuhdo/sample-os
- Интересная статья про программирование модулей для Линукса и про системное программирование https://jvns.ca/blog/2014/09/18/you-can-be-a-kernel-hacker/
- Еще одна статья от автора предыдущей https://jvns.ca/blog/2014/01/04/4-paths-to-being-a-kernel-hacker/
- Пример простого модуля к ядру линукса: https://github.com/jvns/kernel-module-fun/blob/master/hello.c
- Еще один туториал о том, как написать ОС с нуля: https://github.com/cfenollosa/os-tutorial
- Статья "Давайте напишем ядро": https://arjunsreedharan.org/post/82710718100/kernels-101-lets-write-a-kernel
- Сабреддит по разработке ОС: https://www.reddit.com/r/osdev/
- Большой список идей для проектов для разных ЯП, включая C/C++: https://github.com/tuvtran/project-based-learning/blob/master/README.md
- Еще один список идей для проектов https://github.com/danistefanovic/build-your-own-x
- "Давайте напишем ядро с поддержкой ввода с клавиатуры и экрана": https://arjunsreedharan.org/post/99370248137/kernel-201-lets-write-a-kernel-with-keyboard-and
