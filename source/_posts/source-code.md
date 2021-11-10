---
title: 读源码的实用技巧
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
tags:
  - source code
categories:
  - work
abbrlink: 6fff
top:
subtitle:
---

## 读源码的实用技巧

### 1. 查看Java native方法源码

   Java底层的native方法可以在OpenJDK中通过`类名_方法名`（方法名首字母要大写）的形式找到该方法对应的源码实现

   eg：分配直接内存的方法`ByteBuffer.allocateDirect(1024)`源码跟下去会发现核心代码实现是Unsafe.class中的如下方法：

   ```java
private native long allocateMemory0(long var1);
   ```

   想要查看 Unsafe::allocateMemory0(long var1)的源码，可以在OpenJDK中搜索Unsafe_AllocateMemory0，如下：

   ```c
UNSAFE_ENTRY(jlong, Unsafe_AllocateMemory0(JNIEnv *env, jobject unsafe, jlong size)) {
  size_t sz = (size_t)size;

  assert(is_aligned(sz, HeapWordSize), "sz not aligned");

  void* x = os::malloc(sz, mtOther);

  return addr_to_java(x);
} UNSAFE_END
   ```

### 2. 查看操作系统方法介绍

   上述`Unsafe_AllocateMemory0`方法中核心实现方法为`os::malloc(sz, mtOther)`，`malloc`是操作系统的原生方法，此类方法可以借助Linux系统命令`man $方法名 `来查看方法对应的手册条目介绍，如下（不同操作系统可能有略微差别）：

   ```
☁  ~  man malloc
   ```

   ```
MALLOC(3)                BSD Library Functions Manual                MALLOC(3)

NAME
     calloc, free, malloc, realloc, reallocf, valloc, aligned_alloc -- memory allocation

SYNOPSIS
     #include <stdlib.h>

     void *
     calloc(size_t count, size_t size);

     void
     free(void *ptr);

     void *
     malloc(size_t size);
        
        ...

DESCRIPTION
     The malloc(), calloc(), valloc(), realloc(), and reallocf() functions
     allocate memory.  The allocated memory is aligned such that it can be
     used for any data type, including AltiVec- and SSE-related types.  The
     aligned_alloc() function allocates memory with the requested alignment.
     The free() function frees allocations that were created via the preceding
     allocation functions.

     The malloc() function allocates size bytes of memory and returns a
     pointer to the allocated memory.
     ......
   ```
### 3. TODO 持续更新

   