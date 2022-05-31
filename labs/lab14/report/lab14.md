---
## Front matter
title: "Лабораторная работа №14"
subtitle: "Именованные каналы"
author: "Парфенова Елизавета Евгеньевна"

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: PT Serif
romanfont: PT Serif
sansfont: PT Sans
monofont: PT Mono
mainfontoptions: Ligatures=TeX
romanfontoptions: Ligatures=TeX
sansfontoptions: Ligatures=TeX,Scale=MatchLowercase
monofontoptions: Scale=MatchLowercase,Scale=0.9
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Приобретение практических навыков работы с именованными каналами.

# Задание

Изучите приведённые в тексте программы server.c и client.c. Взяв данные примеры
за образец, напишите аналогичные программы, внеся следующие изменения:
1. Работает не 1 клиент, а несколько (например, два).
2. Клиенты передают текущее время с некоторой периодичностью (например, раз в пять
секунд). Используйте функцию sleep() для приостановки работы клиента.
3. Сервер работает не бесконечно, а прекращает работу через некоторое время (например, 30 сек). Используйте функцию clock() для определения времени работы сервера.
Что будет в случае, если сервер завершит работу, не закрыв канал?

# Выполнение лабораторной работы

 Я заранее написала анологичные программы и поместила их в папку lab14. (рис. [-@fig:001])

![Папка с программами](image/1.png){ #fig:001 width=70% }

Пройдемся пр каждму из файлов. 

1. client.c (рис. [-@fig:002])

![client.c](image/2.png){ #fig:002 width=70% }

*Листинг:*

#include "common.h"

#define MESSAGE "Hello Server!!!"

int

main()

{

int msg, len, i;

long int t;

for(i=0; i<20; i++)

{

    sleep(3);
    t=time(NULL);
    printf("FIFO Client...\n");

    if((msg = open(FIFO_NAME, O_WRONLY)) < 0)
    {
        fprintf(stderr,"%s: Невозможно открыть FIFO (%s)\n",
            __FILE__, strerror(errno));
        exit(-1);
    }
    
    len = strlen(MESSAGE);

    if(write(msg, MESSAGE, len) != len)
    {
        fprintf(stderr,"%s: Ошибка записи  в FIFO (%s)\n",
        __FILE__, strerror(errno));
        exit(-2);
    }
    close(msg);
}
    exit(0);
}

2. client2.c (рис. [-@fig:003])

![client2.c](image/3.png){ #fig:003 width=70% }

*Листинг:*

#include "common.h"

#define MESSAGE "Hello Server!!!"

int

main()

{

int writefd, msglen, count;

long long int t;

char message[10];

for(count=0; count<-5; ++count)

{

    sleep(5);
    t=(long long int) time(0);
    sprintf(message, "%lli", t);
    if((writefd = open(FIFO_NAME, O_WRONLY)) < 0)
    {
        fprintf(stderr,"%s: Невозможно открыть FIFO (%s)\n",
            __FILE__, strerror(errno));
        exit(-1);
    }
    
    msglen = strlen(MESSAGE);
    
    if(write(writefd, MESSAGE, msglen) != msglen)
    {
        fprintf(stderr,"%s: Ошибка записи  в FIFO (%s)\n",
        __FILE__, strerror(errno));
        exit(-2);
    }

}
    close(writefd);
    exit(0);
}

3. common.h (рис. [-@fig:004])

![common.h](image/4.png){ #fig:004 width=70% }

*Листинг:*

#ifndef __COMMON_H__

#define __COMMON_H__

#include <stdio.h>

#include <stdlib.h>

#include <string.h>

#include <errno.h>

#include <sys/types.h>

#include <sys/stat.h>

#include <fcntl.h>


#define FIFO_NAME "/tmp/fifo"

#define MAX_BUFF 80

#endif /* __COMMON_H__ */

4. makefile (рис. [-@fig:005])

![makefile](image/5.png){ #fig:005 width=70% }

*Листинг:*

all: server client

server: server.c common.h

		gcc server.c -o server

client: client.c common.h

		gcc client.c -o client

clean:

		-rm server client *.o

5. server.c (рис. [-@fig:006])

![server.c](image/6.png){ #fig:006 width=70% }

*Листинг:*

#include "common.h"

int

main()

{

  int readfd;

  int n;

  char buff[MAX_BUFF];

  printf("FIFO Server...");


  if(mknod(FIFO_NAME, S_IFIFO | 0666, 0) < 0)

    {
      fprintf(stderr, "%s: Невозможно создать FIFO (%s)",
	      __FILE__, strerror(errno));
      exit(-1);
    }

  if((readfd = open(FIFO_NAME, O_RDONLY)) < 0)

    {
      fprintf(stderr, "%s: Невозможно открыть FIFO (%s)",
	      __FILE__, strerror(errno));
      exit(-2);
    }
  clock_t now=time(NULL), start=time(NULL);

  while(now-start<30)

    {
      while((n = read(readfd, buff, MAX_BUFF)) > 0)
	{
	  if(write(1, buff, n) != n)
	    {
	      fprintf(stderr, "%s: Ошибка вывода (%s)",
		      __FILE__, strerror(errno));
	    }
	}
      now=time(NULL);
    }
  printf("server timeout, %li - seconds passed",

  (now-start));

  close(readfd);


  if(unlink(FIFO_NAME) < 0)

    {
      fprintf(stderr, "%s: Невозможно удалить FIFO (%s)",
	      __FILE__, strerror(errno));
      exit(-4);
    }
  exit(0);
    }


Следующим шагом я открыла папку, в которой хранятся все файлы, в терминале и запустила make одноименной командой. (рис. [-@fig:007])

![Запуск make](image/7.png){ #fig:007 width=70% }

Когда все завершилось, в этом же терминале я запустила команду ***./server***. Далее снова открыла эту же папку в терминале (нужно было новое окно) и запустила команду ***./client***. Необходимо оставить оба терминала работающими. Сервер успешно запустился и закончил работу через 30 секунд. (рис. [-@fig:008])

![Работа сервера](image/8.png){ #fig:008 width=70% }


# Выводы

Мы приобрели практические навыки работы с именованными каналами.

# Контрольные вопросы

1. В чем ключевое отличие именованных каналов от неименованных?

Именованные каналы отличаются от неименованных наличием идентификатора канала, который представлен как специальный файл (соответственно имя именованного канала — это имя файла).

2. Возможно ли создание неименованного канала из командной строки?

Создание неименованного канала из командной строки возможно командой pipe.

3. Возможно ли создание именованного канала из командной строки?

Создание именованного канала из командной строки возможно с помощью mkfifo.

4. Опишите функцию языка С, создающую неименованный канал.

Функция языка С, создающая неименованный канал: int read(int pipe_fd, void *area, int cnt); int write(int pipe_fd, void *area, int cnt); Первый аргумент этих вызовов - дескриптор канала, второй - указатель на область памяти, с которой происходит обмен, третий - количество байт. Оба вызова возвращают число переданных байт (или -1 - при ошибке).

5. Опишите функцию языка С, создающую именованный канал.

Функция языка С, создающая именованный канал: int mkfifo (const char *pathname, mode_t mode); Первый параметр — имя файла, идентифицирующего канал, второй параметр маска прав доступа к файлу. Вызов функции mkfifo() создаёт файл канала (с именем, заданным макросом FIFO_NAME): mkfifo(FIFO_NAME, 0600);

6. Что будет в случае прочтения из fifo меньшего числа байтов, чем находится в канале?
Большего числа байтов?

При чтении меньшего числа байтов, возвращается требуемое число байтов, остаток сохраняется для следующих чтений. При чтении большего числа байтов, возвращается доступное число байтов 7. Запись числа байтов, меньшего емкости канала или FIFO, гарантированно атомарно. Это означает, что в случае, когда несколько процессов одновременно записывают в канал, порции данных от этих процессов не перемешиваются. При записи большего числа байтов, чем это позволяет канал или FIFO, вызов write(2) блокируется до освобождения требуемого места. При этом атомарность операции не гарантируется. Если процесс пытается записать данные в канал, не открытый ни одним процессом на чтение, процессу генерируется сигнал SIGPIPE, а вызов write(2) возвращает 0 с установкой ошибки (errno=ЕР1РЕ) (если процесс не установил обработки сигнала SIGPIPE, производится обработка по умолчанию -- процесс завершается).

8. Могут ли два и более процессов читать или записывать в канал?

Два и более процессов могут читать и записывать в канал.

9. Опишите функцию write (тип возвращаемого значения, аргументы и логику работы). Что означает 1 (единица) в вызове этой функции в программе server.c (строка 42)?

Функция write записывает length байтов из буфера buffer в файл, определенный дескриптором файла fd. Эта операция чисто 'двоичная' и без буферизации. При единице возвращает действительное число байтов. Функция write возвращает число действительно записанных в файл байтов или -1 при ошибке, устанавливая при этом errno.

10. Опишите функцию strerror.

Строковая функция strerror - функция языков C/C++, транслирующая код ошибки, который обычно хранится в глобальной переменной errno, в сообщение об ошибке, понятном человеку.