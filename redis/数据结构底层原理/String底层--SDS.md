https://juejin.im/book/5afc2e5f6fb9a07a9b362527/section/5b5af9d96fb9a04f83465ada



# String底层



## 1、字符串结构体 SDS

SDS ：Simple Dynamic String

```C
struct SDS<T> {
  T capacity; // 数组容量 1 byte
  T len; // 数组长度 1 byte
  byte flags; // 特殊标识位，不理睬它 1 byte
  byte[] content; // 数组内容
}
```

底层是一个数组，以 \0 作为结束符号。

Redis规定字符串长度不超过 512M ，创建字符串时 len == capacity ，避免冗余浪费。

扩容时，小于1M成倍扩容，大于1M每次扩容+1M。

## 2、存储方式   [embstr  vs  raw]

embstr : 字符串长度小于44字节时，embstr存储

raw : 大于等于时，采用raw形式存储

### 原理

两种存储方式依赖于Redis内部的内存分配器 jemalloc / tcmalloc .

redis的对象有一个对象头占16字节，SDS本身最少占3个字节，byte[]最后一位又要用 /0 标记结束，那么就有20字节被占用，而内存分配都是2的倍数分配，所以最少要分配32字节，此时是 embstr结构，而一旦字符串内容再多点，就要分配64字节空间，并且剩下可用的是44字节，所以当内容超过44字节时，此时已经是raw的结构了。