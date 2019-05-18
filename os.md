### Что описывае процесс
  - Регистры CPU
  - Состояние памяти (код, данные, стек)
  - Открытые файловые дескрипторы

### Исполнение процесса
  - Код программы
  - Код библиотек
  - Обращение к устройствам через ядро

### Системные вызовы

Переход в пространство ядра осуществляется специальными инструкциями
```
movq $60, $rax  # exit system call
movq $100, $rdi
syscall
```

# Интерфейс системных вызовов Linux

Для перехода используется инструкция `syscall`

Другие варианты: `int $0x80, sysenter, vsyscall, vdso)

Аргументы для syscall передаются на регистрах `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`

Номер систмемногго вызова указывается на `rax`

Результат будет лежать на `rax`

Регистры `rcx` и `r11` не будут восстановлены

### Системные вызовы: open

```
int open(const char* filename, int flags)
```

Возвращает файловый десприктор (`int`)

Десприптор можно использовать для вызовов `read`, `write`

`filename` - путь на файловой системе. Абсолютный от корня `/`, относительный от текущей директории.

Файл может быть как обычным, так и устройством.

```
# /dev/tty - текущая консоль
echo "hello" > /dev/tty
```

### Системные вызовы: read, write

```
ssize_t read(int fd, void *buf, size_t count)
ssize_t write(int fd, void *buf, size_t count)
```

`read` читает в буфер данные из файла

`write` записывает в файл данные из буфера

`read/write` меняет текущую позицию в файловом объекте

вызов `read/write` блокируется

### Системные вызовы: close

```
int close(int fd)
```

Закрывает файловый дескриптор. Дескриптор освобождается и может быть использован для вновь открытых файлов.

Если дескриптор был последним указывающим на файловый объект, то объект освобождается.

Если дескриптор был последним указателем на удаленный файл, то файл удаляется оканчательно.

### Системные вызовы: fork

```
pid_t fork()
```

Создает копию исходного процесса.

Память и регистры - копия (за исключением кода возврата из `fork`)

Файловые дескрипторы - копия, причем они ссылаются на те же файловые объекты в ядре.

У родителя и потомка общие файлы, общие указатели на позиции в файлах.


### Системные вызовы: exec

```
int execve(const char *filename, char *const argv[], char * const envp[])
```

Загружает новый образ в текущем процессе

Образ - `filename` или интерпритатор (для скриптов начинающихся с `#!/bin/sh`)

`argv` - аргументы для передачи `main`

`envp` - переменные окружения (доступны если объявить `main(int ac, char* av[], char* ep[]))

В библиотеке есть другие сигнатуры

### Системные вызовы: waitpid

```
pid_t waitpid(pit_t pid, int *wstatus, int options)
```

Ждет когда завершится
  - `pid = -1` любой поток
  - `pid > 0` поток с конкртным `pid`

В библиотеке есть другие сигнатуры

### Системные вызовы: pipe

```
int pipe(int pipefd[2])
```

Результатом вызова будут два дескриптора
  - `pipe[0]` для чтения
  - `pipe[1]` для записи

Код возврата нужен для сообщений об ошибке

Unix позволяет перенаправлять `stdout` одного процесса в `stdin` другого

В основе лежит конструкция `pipe`

`pipe` - буфер ядра, в который можно писать под одному дескриптору и читать из другого

### Системные вызовы для выделения памяти

`malloc` - функция библиотеки `C`, а не системный вызов.

`brk`, `sbrk` - подвинуть границу сегмента данных.

`mmap` - в Linux так можно выделять память указав флаг `MAP_ANONYMOUS`

Современные аллокаторы работают через `mmap`.

### Системные вызовы для работы с ФС

`open` с флагом `0_CREAT` создаст файл

`chdir` поменяет текущую директорию

`mkdir` создаст новую директорию

`mknod` создаст специальный файл. Так создаются файлы устройств в `/dev`

`link` создаст новую жесткую ссылку на файл

`unlink` удалить ссылку на файл. Если на файл больше нет ссылок и нет открытых дескрипторов, он удалится.

### Состояния процесса xv6

- EMBRYO -> [RUNNABLE]
- RUNNABLE -> [RUNNING]
- RUNNING -> [RUNNABLE, ZOMBIE, SLEEPIN]
- ZOMBIE
- SLEEPING -> [RUNNABLE]