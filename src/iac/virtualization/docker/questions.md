### Что такое Docker? В чем отличие контейнера от образа?

Docker — это инструмент, позволяющий запускать приложения в изолированных контейнерах.

Образ — шаблон приложения, который содержит слои файловой системы в режиме "только-чтение".

Контейнер — запущенный образ приложения, который кроме нижних слоев в режиме "только чтение" содержит верхний слой в режиме "чтение-запись".

### Какие инструкции есть у Dockerfile?

| Инструкция | Описание |
|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `FROM` | Задаёт базовый (родительский) образ. |
| `LABEL` | Описывает метаданные. Например — сведения о том, кто создал и поддерживает образ. |
| `ENV` | Устанавливает постоянные переменные среды. |
| `RUN` | Выполняет команду и создаёт слой образа. Используется для установки в контейнер пакетов. |
| `COPY` | Копирует в контейнер файлы и директории. |
| `ADD` | Копирует файлы и директории в контейнер, может распаковывать локальные .tar-файлы. |
| `CMD` | Описывает команду с аргументами, которую нужно выполнить когда контейнер будет запущен. Аргументы могут быть переопределены при запуске контейнера. В файле может присутствовать лишь одна инструкция CMD. |
| `WORKDIR` | Задаёт рабочую директорию для следующей инструкции. |
| `ARG` | Задаёт переменные для передачи Docker во время сборки образа. |
| `ENTRYPOINT` | Предоставляет команду с аргументами для вызова во время выполнения контейнера. Аргументы не переопределяются. |
| `EXPOSE` | Указывает на необходимость открыть порт. |
| `VOLUME` | Создаёт точку монтирования для работы с постоянным хранилищем. |

### Чем отличается `CMD` от `ENTRYPOINT` в Dockerfile?

Инструкции `CMD` и `ENTRYPOINT` выполняются в момент запуска контейнера, только инструкция `CMD` позволяет переопределить передаваемые команде аргументы.

**Пример 1. `CMD`:**
Опишем сборку образа в Dockerfile.
```
FROM alpine  
CMD ["ping", "8.8.8.8"]  
```
В инструкцию `CMD` передаются 2 аргумента. Выполним сборку образа `docker build -t test .` и запустим контейнер.
```shell
$ docker run test
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=43 time=32.976 ms
64 bytes from 8.8.8.8: seq=1 ttl=43 time=31.998 ms
64 bytes from 8.8.8.8: seq=2 ttl=43 time=31.843 ms
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 31.708/33.316/36.823 ms
```
Теперь передадим 2 новых аргумента для запуска контейнера.
```shell
$ docker run test traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 46 byte packets
 1  172.17.0.1 (172.17.0.1)  0.017 ms  0.016 ms  0.009 ms
 2  192.168.168.1 (192.168.168.1)  0.996 ms  1.553 ms  2.069 ms
 3  *  *  *
 4  lag-2-435.bgw01.samara.ertelecom.ru (85.113.62.125)  1.454 ms  1.427 ms  1.984 ms
 5  172.68.8.3 (172.68.8.3)  19.685 ms  15.722 ms  15.565 ms
 6  172.68.8.2 (172.68.8.2)  15.846 ms  22.696 ms  35.093 ms
 7  one.one.one.one (1.1.1.1)  17.439 ms  17.670 ms  24.202 ms
```
`ping` заменен на traceroute, IP адрес заменен на 1.1.1.1.

**Пример 2. `ENTRYPOINT`:**
Опишем сборку образа в Dockerfile.
```dockerfile
FROM alpine  
ENTRYPOINT ["ping", "8.8.8.8"]
```
В инструкцию `ENTRYPOINT` передаются 2 аргумента. Выполним сборку образа `docker build -t test .` и запустим контейнер.
```shell
$ docker run test2
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=43 time=36.189 ms
64 bytes from 8.8.8.8: seq=1 ttl=43 time=44.120 ms
64 bytes from 8.8.8.8: seq=2 ttl=43 time=44.584 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 36.189/41.631/44.584 ms
```
Теперь передадим и изменим один из аргументов для запуска контейнера.
```shell
$ docker run test2 ping 1.1.1.1
BusyBox v1.31.1 () multi-call binary.

Usage: ping [OPTIONS] HOST

Send ICMP ECHO_REQUEST packets to network hosts

	-4,-6		Force IP or IPv6 name resolution
	-c CNT		Send only CNT pings
	-s SIZE		Send SIZE data bytes in packets (default 56)
	-i SECS		Interval
	-A		Ping as soon as reply is recevied
	-t TTL		Set TTL
	-I IFACE/IP	Source interface or IP address
	-W SEC		Seconds to wait for the first response (default 10)
			(after all -c CNT packets are sent)
	-w SEC		Seconds until ping exits (default:infinite)
			(can exit earlier with -c CNT)
	-q		Quiet, only display output at start
			and when finished
	-p HEXBYTE	Pattern to use for payload
```
Как видим, аргумент передать контейнеру нельзя.

**Пример 3. `ENTRYPOINT` и `CMD`:**
Опишем сборку образа в Dockerfile.
```dockerfile
FROM alpine  
ENTRYPOINT ["ping"]
CMD ["8.8.8.8"]
```
В инструкцию `ENTRYPOINT` передаётся аргумент `ping`, в CMD передаётся аргумент 8.8.8.8. Выполним сборку образа `docker build -t test .` и запустим контейнер.
```shell
$ docker run test3
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=43 time=41.176 ms
64 bytes from 8.8.8.8: seq=1 ttl=43 time=32.875 ms
64 bytes from 8.8.8.8: seq=2 ttl=43 time=40.395 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 32.875/38.148/41.176 ms
```
Пробуем изменить 2 аргумента.
```shell
$ docker run test3 traceroute 1.1.1.1
BusyBox v1.31.1 () multi-call binary.

Usage: ping [OPTIONS] HOST

Send ICMP ECHO_REQUEST packets to network hosts

	-4,-6		Force IP or IPv6 name resolution
	-c CNT		Send only CNT pings
	-s SIZE		Send SIZE data bytes in packets (default 56)
	-i SECS		Interval
	-A		Ping as soon as reply is recevied
	-t TTL		Set TTL
	-I IFACE/IP	Source interface or IP address
	-W SEC		Seconds to wait for the first response (default 10)
			(after all -c CNT packets are sent)
	-w SEC		Seconds until ping exits (default:infinite)
			(can exit earlier with -c CNT)
	-q		Quiet, only display output at start
			and when finished
	-p HEXBYTE	Pattern to use for payload
```
Изменить 2 аргумента невозможно. Заменим аргумент инструкции `CMD`.
```shell
$ docker run test3 1.1.1.1    
PING 1.1.1.1 (1.1.1.1): 56 data bytes
64 bytes from 1.1.1.1: seq=0 ttl=58 time=31.412 ms
64 bytes from 1.1.1.1: seq=1 ttl=58 time=19.400 ms
64 bytes from 1.1.1.1: seq=2 ttl=58 time=15.814 ms
^C
--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 15.814/22.208/31.412 ms
```
При такой сборке образа команды `ENTRYPOINT` и `CMD` при запуске контейнера будут запущены последовательно, но аргумент возможно изменить только для `CMD`.

### Чем отличается *`COPY`* от *`ADD`* в Dockerfile?

`COPY` копирует файлы и папки с хоста в контейнер. `ADD` может делать то же самое, но ещё распаковывать архивы.

### Какие есть best practices для написания Dockerfile?

1. Запускать только один процесс на контейнер.
2. Стараться объединять несколько команд `RUN` в одну для уменьшения количества слоёв образа.
3. Часто изменяемые слои образа необходимо располагать ниже по уровню, чтобы ускорить процесс сборки, т.к. при изменении верхнего слоя, все нижеследующие слои будут пересобираться.
4. Указывать явные версии образов в инструкции `FROM`, чтобы избежать случая, когда выйдет новая версия образа с тегом `latest`.
5. При установке пакетов указывать версии пакетов.
6. Очищать кеш пакетного менеджера и удалять ненужные файлы после выполненной инструкции.
7. Использовать `multistage build` для сборки артефакта в одном контейнере и размещении его в другом.