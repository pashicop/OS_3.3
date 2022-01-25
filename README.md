# OS_3.3
# 1
```
vagrant@vagrant:~$ strace /bin/bash -c cd t1 2>&1 | grep cd
execve("/bin/bash", ["/bin/bash", "-c", "cd", "t1"], 0x7ffe0819f198 /* 24 vars */) = 0
```
# 2
Файл `/etc/magic`
```
vagrant@vagrant:~$ strace file /dev/tty 2>&1
execve("/usr/bin/file", ["file", "/dev/tty"], 0x7ffe23ed13a8 /* 24 vars */) = 0
brk(NULL)                               = 0x5590336ce000
...
stat("/home/vagrant/.magic.mgc", 0x7fff90e6a990) = -1 ENOENT (No such file or directory)
stat("/home/vagrant/.magic", 0x7fff90e6a990) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
**openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3**
fstat(3, {st_mode=S_IFREG|0644, st_size=111, ...}) = 0
read(3, "# Magic local data for file(1) c"..., 4096) = 111
read(3, "", 4096)                       = 0
close(3)                                = 0
...
exit_group(0)                           = ?
+++ exited with 0 +++
vagrant@vagrant:~$
```
# 3
# 6

```
vagrant@vagrant:/etc$ strace uname -a
execve("/usr/bin/uname", ["uname", "-a"], 0x7ffd6288bda8 /* 24 vars */) = 0
...
**uname({sysname="Linux", nodename="vagrant", ...}) = 0**
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
uname({sysname="Linux", nodename="vagrant", ...}) = 0
write(1, "Linux vagrant 5.4.0-91-generic #"..., 106Linux vagrant 5.4.0-91-generic #102-Ubuntu SMP Fri Nov 5 16:31:28 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
) = 106
...
+++ exited with 0 +++
```
Можно посмотреть в `/proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}`
`Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, **osrelease**, **version**, domainname}.`
# 7
С `;` команды выполняются последовательно независимо от результата предыдущей
Команда за `&&` выполняется, если предыдущая выполнилась с exit-кодом 0
Не имеет смысла использовать `&&` при использовании до `set -e`, так как команда завершается сразу, если exit-код не 0. В моём случае завершалось ssh подключение при завершении команды set с ошибками, т.е. exit-код не 0
```
set -e test=&& && echo 3
-bash: syntax error near unexpected token `&&'
Connection to 127.0.0.1 closed.
```
# 8
`e` завершает работу сразу, если exit код не 0
`u` считать не установленные переменные как ошибку при подстановке
`x` отображать команды и их аргументы в порядке из выполнения/использования
`o pipefail` exit код pipeline будет 0, если все команды завершились с 0 статусом, и будет равен последнему ненулевому exit коду команды
Удобно смотреть вывод скрипта, отлаживать скрипт
# 9
Больше всего процессов с состоянием `S` (interruptible sleep)
```
vagrant@vagrant:~$ vagrant@vagrant:~$ ps axo stat|grep -c S
63
vagrant@vagrant:~$ ps axo stat|grep -c I
49
```

```
               <    high-priority (not nice to other users)
               N    low-priority (nice to other users)
               L    has pages locked into memory (for real-time and custom IO)
               s    is a session leader
               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
               +    is in the foreground process group
```
