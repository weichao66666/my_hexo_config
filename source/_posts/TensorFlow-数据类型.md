---
layout: article
title: TensorFlow 数据类型
date: 2019-05-06 09:23:38
tags:
categories: 
copyright: true
---

# **Reference**

* [深度学习与 TensorFlow 2 入门实战](https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469 "https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469")
* [TensorFlow-2.x-Tutorials](https://github.com/dragen1860/TensorFlow-2.x-Tutorials "https://github.com/dragen1860/TensorFlow-2.x-Tutorials")

---

# **list**

```python
[1, 1.2, 'Hello', (1, 2), layers]
[64, 32, 32, 3]
```

---

# **np.array**

```python
[64, 224, 224, 3]
```

解决同类型数据运算。

深度计算之前已经设计好的科学计算库，不支持 GPU，不支持自动求导。TensorFlow 解决了这一问题。

---

# **tf.Tensor**

## **int, float, double, bool, string**

```python
a = tf.constant(1)
print(a) # tf.Tensor(1, shape=(), dtype=int32)

a = tf.constant(1.)
print(a) # tf.Tensor(1.0, shape=(), dtype=float32)

a = tf.constant(2.2, dtype=tf.int32)
print(a) # TypeError: Cannot convert provided value to EagerTensor. Provided value: 2.2 Requested dtype: int32

a = tf.constant(2.2, dtype=tf.double)
print(a) # tf.Tensor(2.2, shape=(), dtype=float64)

a = tf.constant([True, False])
print(a) # tf.Tensor([ True False], shape=(2,), dtype=bool)

a = tf.constant('Hello, world!')
print(a) # tf.Tensor(b'Hello, world!', shape=(), dtype=string)
```

## **scalar**

```python
1.1
```

## **vector**

```python
[1.1], [1.1, 1.2]
```

## **matrix**

```python
[[1.1, 2.2], [3.3, 4.4], [5.5, 6.6]]
```

## **tensor**

rank > 2

---

# **Tensor Property**

## **切换 Tensor 到 CPU 或 GPU**

```python
with tf.device("cpu"):
    a = tf.constant([1])
with tf.device("gpu"):
    b = tf.range(4)
print(a.device) # /job:localhost/replica:0/task:0/device:CPU:0
print(b.device) # /job:localhost/replica:0/task:0/device:GPU:0

aa = a.gpu()
bb = b.cpu()
print(aa.device) # /job:localhost/replica:0/task:0/device:GPU:0
print(bb.device) # /job:localhost/replica:0/task:0/device:CPU:0
```

## **Tensor 转 numpy**

```python
b = tf.range(4)
bb = b.numpy()
print(b) # tf.Tensor([0 1 2 3], shape=(4,), dtype=int32)
print(bb) # [0 1 2 3]
```

## **numpy 转 Tensor**

```python
a = np.arange(5)
a1 = a.dtype
a2 = tf.convert_to_tensor(a)
a3 = tf.convert_to_tensor(a, dtype=tf.int64)
print(a1) # int32
print(a2) # tf.Tensor([0 1 2 3 4], shape=(5,), dtype=int32)
print(a3) # tf.Tensor([0 1 2 3 4], shape=(5,), dtype=int64)
```

## **int32 转 float32/double/int64**

```python
b = tf.range(4)
b1 = tf.cast(b, dtype=tf.float32)
b2 = tf.cast(b, dtype=tf.double)
b3 = tf.cast(b, dtype=tf.int64)
print(b) # tf.Tensor([0 1 2 3], shape=(4,), dtype=int32)
print(b1) # tf.Tensor([0. 1. 2. 3.], shape=(4,), dtype=float32)
print(b2) # tf.Tensor([0. 1. 2. 3.], shape=(4,), dtype=float64)
print(b3) # tf.Tensor([0 1 2 3], shape=(4,), dtype=int64)

a = tf.ones([])
a1 = int(a)
a2 = float(a)
print(a) # tf.Tensor(1.0, shape=(), dtype=float32)
print(a1) # 1
print(a2) # 1.0
```

## **int32 和 bool 的相互转换**

```python
b = tf.constant([0, 1])
b1 = tf.cast(b, dtype=tf.bool)
b2 = tf.cast(b1, dtype=tf.int32)
print(b1) # tf.Tensor([False  True], shape=(2,), dtype=bool)
print(b2) # tf.Tensor([0 1], shape=(2,), dtype=int32)
```

## **Tensor 转 Variable**

```python
a = tf.range(5)
b = tf.Variable(a)
print(b.trainable) # True

b1 = isinstance(b, tf.Tensor)
b2 = isinstance(b, tf.Variable)
b3 = tf.is_tensor(b)
print(b1) # False
print(b2) # True
print(b3) # True
```

## **获取 Tensor 的 rank**

```python
a = tf.constant(1.1)
b = tf.constant([1.1])
print(a.ndim) # 0
print(b.ndim) # 1
print(tf.rank(a)) # tf.Tensor(0, shape=(), dtype=int32)
print(tf.rank(b)) # tf.Tensor(1, shape=(), dtype=int32)
```

## **判断是否为 Tensor**

```python
a = tf.constant([1.])
d = np.arange(4)
a1 = isinstance(a, tf.Tensor)
a2 = tf.is_tensor(a)
d1 = isinstance(d, tf.Tensor)
d2 = tf.is_tensor(d)
print(a1) # True
print(a2) # True
print(d1) # False
print(d2) # False
```

## **获取 Tensor 的数据类型**

```python
a = tf.constant([1.])
d = np.arange(4)
print(a.dtype) # <dtype: 'float32'>
print(d.dtype) # int32
```

## **判断 Tensor 是否为指定的数据类型**

```python
a = tf.constant([1.])
d = np.arange(4)
print(a.dtype == tf.float32) # True
print(d.dtype == np.int32) # True
```

---

# **Tensor 初始化**

## **from numpy, list**

```python
t1 = tf.convert_to_tensor(np.ones([2, 3]))
t2 = tf.convert_to_tensor([2, 3])
print(t1) # tf.Tensor(
          # [[1. 1. 1.]
          #  [1. 1. 1.]], shape=(2, 3), dtype=float64)
print(t2) # tf.Tensor([2 3], shape=(2,), dtype=int32)
```

## **zeros**

```python
t1 = tf.zeros([2, 3])
print(t1) # tf.Tensor(
          # [[0. 0. 0.]
          #  [0. 0. 0.]], shape=(2, 3), dtype=float32)
```

## **ones**

```python
t1 = tf.ones([2, 3])
print(t1) # tf.Tensor(
          # [[1. 1. 1.]
          #  [1. 1. 1.]], shape=(2, 3), dtype=float32)
```

## **fill**

```python
t1 = tf.fill([2, 3], 3.14)
print(t1) # tf.Tensor(
          # [[3.14 3.14 3.14]
          #  [3.14 3.14 3.14]], shape=(2, 3), dtype=float32)
```

## **random**

### **normal, truncated_normal**

```python
t1 = tf.random.normal([2, 3], mean=1, stddev=1)
t2 = tf.random.normal([2, 3], mean=1, stddev=1)
t3 = tf.random.truncated_normal([2, 3], mean=1, stddev=1)
print(t1) # tf.Tensor(
          # [[1.2308737 0.8250756 2.6446304]
          #  [2.714213  2.0199542 1.3591613]], shape=(2, 3), dtype=float32)
print(t2) # tf.Tensor(
          # [[ 1.5890408  -0.8608984   1.7205659 ]
          #  [ 1.9205048   0.7741975   0.73170424]], shape=(2, 3), dtype=float32)
print(t3) # tf.Tensor(
          # [[ 1.3489898 -0.4739654  0.5768948]
          #  [ 2.3000512  0.7400613  1.8497746]], shape=(2, 3), dtype=float32)
```

### **uniform**

```python
t1 = tf.random.uniform([2, 3], minval=0, maxval=100)
print(t1) # tf.Tensor(
          # [[19.821571  44.24106   29.511225 ]
          #  [ 0.7015109  7.66474    5.251384 ]], shape=(2, 3), dtype=float32)
```

### **shuffle**

```python
id = tf.range(10)
id1 = tf.random.shuffle(id)
print(id) # tf.Tensor([0 1 2 3 4 5 6 7 8 9], shape=(10,), dtype=int32)
print(id1) # tf.Tensor([8 1 4 0 7 2 5 3 9 6], shape=(10,), dtype=int32)
```

## **constant**

```python
t1 = tf.constant(1)
print(t1) # tf.Tensor(1, shape=(), dtype=int32)
```

## **range**

```python
t1 = tf.range(4)
t2 = tf.one_hot(t1, 6)
print(t1) # tf.Tensor([0 1 2 3], shape=(4,), dtype=int32)
print(t2) # tf.Tensor(
          # [[1. 0. 0. 0. 0. 0.]
          #  [0. 1. 0. 0. 0. 0.]
          #  [0. 0. 1. 0. 0. 0.]
          #  [0. 0. 0. 1. 0. 0.]], shape=(4, 6), dtype=float32)
```

---


