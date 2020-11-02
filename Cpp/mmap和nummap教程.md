# mmap和munmap教程



## 语法

```c++
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
int munmap(void *addr, size_t length);
```



## mmap

```c++
void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
```



mmap将一个文件或者其它对象映射进内存。文件被映射到多个页上，如果文件的大小不是所有页的大小之和，最后一个页不被使用的空间将会清零。

- `addr`: 指定需要映射内存的开始地址，如果`addr`为null，则由系统内核指定。
- `length`:指定所要映射的长度
- `prot`: 指定对这块内存的权限。`PROT_EXEC`可执行，`PROT_READ`可读，`PROT_WRITE`可写，`PROT_NONE`不可访问
- `flags`：指定是否与其他进程共享这块内存。

| flag                  | 描述                                                   |
| --------------------- | ------------------------------------------------------ |
| `MAP_SHARED`          | 其他进程可访问                                         |
| `MAP_SHARED_VALIDATE` | 与`MAP_SHARED`大致一样，不过在其他进程访问时需要验证。 |
| `MAP_PRIVATE`         | 其他进程不可见                                         |

- `fd`:文件描述符
- `offset`:偏移量，一般是page size的整数倍





## munmap

删除指定范围内的内存映射。addr必须是page size的整数倍。





