---
id: adding-models
title: 模块
layout: docs
permalink: /docs/adding-models.html
section: User Guide
section_order: 01
order: 07
---

## 为什么需要模块

当我们分析一个项目时， 函数之间相互依赖。Infer 会跟随调用关系决定以什么样的一个顺序分析这些函数。这样做的目的是不管在哪里调用该方法时，总能用上这个方法的分析概要。

我们举个例子来说明：


```C
int foo(int x) {
  if (x < 42) {
    return x;
  } else {
    return 0;
  }
}

int bar() {
  return foo(24);
}

int baz() {
  return foo(54);
}
```

从 `foo` 开始，Infer 发现这个函数要么在参数小于 `42` 时返回 `0`，要么直接返回参数的值。根据这个信息，Infer 便可以发现，`bar` 总是返回 `24` 而 `baz` 总是返回 `0`。

当然，在分析的过程中，一些函数的代码可能不存在。比如，一个使用预编译库的项目，最典型的情况就是我们是一共标准库的时候，如下：

```C
#include <stdlib.h>

int* create() {
  int *p = malloc(sizeof(int));
  if (p == NULL) exit(1);
  return p;
}

void assign(int x, int *p) {
  *p = x;
}

int* my_function() {
  int *p = create();
  assign(42, p);
  return p;
}
```

在上面的例子中，当 Infer 首先会分析 `create` 函数，当这时 `malloc` 对应的源码是找不到的。为了处理这种情况，Infer 依赖这些没有源码的函数的模块才能进行分析。`malloc` 函数在内部模块化为要么返回 `NULL` 要么返回一个有效的分配后的内存指针。类似地，`exit` 函数被模块化为退出执行。使用这两个模型，Infer 发现 `create` 总是发现一个分配后的指针，`my_function` 是安全的。

这时，我们必须要注意，缺失源码和模块都不会导致分析失败。源码缺失的函数被当做无效函数。虽然在大多数情况下，会直接跳过这些函数不会有问题，但这有可能影响到分析的质量。比如缺失的模块将有可能导致生成不正确的 bug 报告。

我们现在来考虑这样一种情况：有一个函数 `lib_exit` 和 `exit` 有相同的作用，不过这个函数被定义在一个不属于这个项目的预编译库中。

```C
void lib_exit(int);

int* create() {
  int *p = malloc(sizeof(int));
  if (p == NULL) lib_exit(1);
  return p;
}
```

这时，Infer 只有在 `p` 非空的时候，才有可能知道返回值。当分析 `my_function` 时，Infer 会考虑参数为空的情况，并在调用 `assign(42, p)` 处报一个空引用错误。

相同地， 我们有 `lib_alloc` 和 `malloc`，`create` 定义如下：

```C
int* lib_alloc(int);

int* create() {
  int *p = lib_alloc(sizeof(int));
  return p;
}
```

这时，在 `my_function` 却不会报任何空引用错误。因为 lib_alloc 源码缺失，返回总是 `NULL`。

## 示例

### C

添加新的模块很简单，C 的模块在 [`infer/models/c/src/`](https://github.com/facebook/infer/tree/master/infer/models/c/src). The file [`libc_basic.c`](https://github.com/facebook/infer/blob/master/infer/models/c/src/libc_basic.c)。这包含了为 C 标准库中最常用的函数定义的模块。 比如，`xmalloc` ，本质上和上述 `create` 是一样的。模块定义如下：

```C
void *xmalloc(size_t size) {
  void *ret = malloc(size);
  INFER_EXCLUDE_CONDITION(ret == NULL);
  return ret;
}
```

The function `xmalloc` is modeled using `malloc` to create an allocated object and the macro `INFER_EXCLUDE_CONDITION` used to eliminate the case where `malloc` can return null. The list of helper functions and macros for writing models can be found in [`infer_builtins.c`](https://github.com/facebook/infer/blob/master/infer/models/c/src/infer_builtins.c).

For a slightly more complex example, `realloc` is modeled as:

```C
void *realloc(void *ptr, size_t size) {
  if(ptr==0) { // if ptr in NULL, behave as malloc
    return malloc(size);
  }
  int old_size;
  int can_enlarge;
  old_size = __get_array_size(ptr); // force ptr to be an array
  can_enlarge = __infer_nondet_int(); // nondeterministically choose whether the current block can be enlarged
  if(can_enlarge) {
    __set_array_size(ptr, size); // enlarge the block
    return ptr;
  }
  int *newblock = malloc(size);
  if(newblock) {
    free(ptr);
    return newblock;
  }
  else { // if new allocation fails, do not free the old block
    return newblock;
  }
}
```

This model is based on existing models for `malloc` and `free` and three helper functions:

- `__get_array_size(ptr)` which allows to manipulate with a model what Infer knows about the size of the allocated memory
- `__set_array_size(ptr, size)` to modify the information about the size of the allocated memory
- `__infer_nondet_int()` to create a variable which can have any possible integer value

### For Java

The models for Java are following the same approach and the list of helper functions is in:

  [`infer/models/java/src/com/facebook/infer/models/InferBuiltins.java`](https://github.com/facebook/infer/blob/master/infer/models/java/src/com/facebook/infer/models/InferBuiltins.java)
  [`infer/models/java/src/com/facebook/infer/models/InferUndefined.java`](https://github.com/facebook/infer/blob/master/infer/models/java/src/com/facebook/infer/models/InferUndefined.java)

For example, Infer treats Java hash maps using a recency abstraction model: Infer remembers the last two keys being added by `put` and checked by `containsKey`, which can be used to make sure that no null pointer exceptions are coming from the fact that `get(key)` returns null when `key` is not not in the map. This behavior can just be implemented via a model written in Java with the help of few helper functions understood by Infer. These models can be found in:

  [`infer/models/java/src/java/util/HashMap.java`](https://github.com/facebook/infer/blob/master/infer/models/java/src/java/util/HashMap.java)

and just rely on these two methods:

- `InferUndefined.boolean_undefined()` to create a non-deterministic choice
- `(V)InferUndefined.object_undefined()` to create a non null undefined object of type `V`

## How to add new models

Let's look at a toy example in Java. As explained above, models for C, Objective-C and Java are all following the same approach.

```Java
import lib.Server;

public class Test {

  enum Status {
    SUCCESS, FAILURE, PING_FAILURE, CONNECTION_FAILURE
  }

  Status convertStatus(Server s) {
    switch (s.getStatus()) {
    case 0:
      return Status.SUCCESS;
    case 1:
      return Status.FAILURE;
    case 2:
      return Status.FAILURE;
    default: // should not happen
      return null;
    }
  }

  String statusName(Server s) {
    Status status = convertStatus(s);
    return status.name();
  }

}
```

Assuming that the class `lib.Server` is part of a pre-compiled library, Infer will report a null pointer exception in `statusName`. This happens whenever `s.getStatus()` returns a value greater that `3`, in which case the default branch of the switch statement is taken and `convertStatus` returns `null`. However, we know from the documentation that the method `lib.Server.getStatus` can only return `0`, `1`, or `2`. A possible approach would be to use an assertion like the Guava `Preconditions.checkState` to inform Infer about the invariant:

```Java
Status convertStatus(Server s) {
  int serverStatus = s.getStatus();
  Preconditions.checkState(serverStatus >= 0 && serverStatus < 3);
  switch (s.getStatus()) {
    ...
  }
}
```

However, in the case where adding preconditions is not possible, we can then write a model for `getStatus()` in order to make the analysis more precise.

To create a model for `getStatus()`, we need to add a class with the name and the same package as for the original method. In this example:

- create a file `infer/models/java/src/infer/models/Server.java` with the following content:

    ```Java
    package infer.models;

    import com.facebook.infer.models.InferBuiltins;
    import com.facebook.infer.models.InferUndefined;

    public class Server {

      public int getStatus() {
        int status = InferUndefined.int_undefined();
        InferBuiltins.assume(status >= 0 && status < 3);
        return status;
      }
    }
    ```

- recompile infer:

    ```bash
    make -C infer
    ```

- run the analysis again:

    ```bash
    infer -- javac Test.java
    ```

Now it should no longer report a null pointer exception.
