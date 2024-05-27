# ConcurrentHashMap 1.7

## 存储结构

![alt text](image.png)

$Java 7$ 中 `ConcurrentHashMap` 的存储结构如上图， `ConcurrentHashMap` 由很多个 `Segment` 组合，而每一个 `Segment` 是一个类似于 `HashMap` 的结构，所以每一个 `