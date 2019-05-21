---
layout: article
title: TensorFlow 合并与分割
date: 2019-05-21 17:00:12
tags:
categories: 
copyright: true
---

# **Reference**

* [深度学习与 TensorFlow 2 入门实战](https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469 "https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469")
* [TensorFlow-2.x-Tutorials](https://github.com/dragen1860/TensorFlow-2.x-Tutorials "https://github.com/dragen1860/TensorFlow-2.x-Tutorials")

---

# **concat**

```python
a = tf.range(12)
b = tf.range(12, 36)
a1 = tf.reshape(a, [2, 6])
b1 = tf.reshape(b, [4, 6])
t = tf.concat([a1, b1], axis=0)
print(a) # tf.Tensor([ 0  1  2  3  4  5  6  7  8  9 10 11], shape=(12,), dtype=int32)
print(b) # tf.Tensor([12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35], shape=(24,), dtype=int32)
print(a1) # tf.Tensor(
          # [[ 0  1  2  3  4  5]
          #  [ 6  7  8  9 10 11]], shape=(2, 6), dtype=int32)
print(b1) # tf.Tensor(
          # [[12 13 14 15 16 17]
          #  [18 19 20 21 22 23]
          #  [24 25 26 27 28 29]
          #  [30 31 32 33 34 35]], shape=(4, 6), dtype=int32)
print(t) # tf.Tensor(
         # [[ 0  1  2  3  4  5]
         #  [ 6  7  8  9 10 11]
         #  [12 13 14 15 16 17]
         #  [18 19 20 21 22 23]
         #  [24 25 26 27 28 29]
         #  [30 31 32 33 34 35]], shape=(6, 6), dtype=int32)
```

---

# **stack**

```python
a = tf.range(12)
c = tf.range(12, 24)
a1 = tf.reshape(a, [2, 6])
c1 = tf.reshape(c, [2, 6])
t = tf.stack([a1, c1], axis=0)
print(a) # tf.Tensor([ 0  1  2  3  4  5  6  7  8  9 10 11], shape=(12,), dtype=int32)
print(c) # tf.Tensor([12 13 14 15 16 17 18 19 20 21 22 23], shape=(12,), dtype=int32)
print(a1) # tf.Tensor(
          # [[ 0  1  2  3  4  5]
          #  [ 6  7  8  9 10 11]], shape=(2, 6), dtype=int32)
print(c1) # tf.Tensor(
          # [[12 13 14 15 16 17]
          #  [18 19 20 21 22 23]], shape=(2, 6), dtype=int32)
print(t) # tf.Tensor(
         # [[[ 0  1  2  3  4  5]
         #   [ 6  7  8  9 10 11]]
         # 
         #  [[12 13 14 15 16 17]
         #   [18 19 20 21 22 23]]], shape=(2, 2, 6), dtype=int32)
```

---

# **unstack**

```python
a = tf.range(12)
c = tf.range(12, 24)
a1 = tf.reshape(a, [2, 6])
c1 = tf.reshape(c, [2, 6])
t = tf.stack([a1, c1], axis=0)
t1 = tf.unstack(t, axis=0)
t2 = tf.unstack(t, axis=2)
print(a) # tf.Tensor([ 0  1  2  3  4  5  6  7  8  9 10 11], shape=(12,), dtype=int32)
print(c) # tf.Tensor([12 13 14 15 16 17 18 19 20 21 22 23], shape=(12,), dtype=int32)
print(a1) # tf.Tensor(
          # [[ 0  1  2  3  4  5]
          #  [ 6  7  8  9 10 11]], shape=(2, 6), dtype=int32)
print(c1) # tf.Tensor(
          # [[12 13 14 15 16 17]
          #  [18 19 20 21 22 23]], shape=(2, 6), dtype=int32)
print(t) # tf.Tensor(
         # [[[ 0  1  2  3  4  5]
         #   [ 6  7  8  9 10 11]]
         # 
         #  [[12 13 14 15 16 17]
         #   [18 19 20 21 22 23]]], shape=(2, 2, 6), dtype=int32)
print(t1) # [<tf.Tensor: id=13, shape=(2, 6), dtype=int32, numpy=
          # array([[ 0,  1,  2,  3,  4,  5],
          #        [ 6,  7,  8,  9, 10, 11]])>, <tf.Tensor: id=14, shape=(2, 6), dtype=int32, numpy=
          # array([[12, 13, 14, 15, 16, 17],
          #        [18, 19, 20, 21, 22, 23]])>]
print(t2) # [<tf.Tensor: id=15, shape=(2, 2), dtype=int32, numpy=
          # array([[ 0,  6],
          #        [12, 18]])>, <tf.Tensor: id=16, shape=(2, 2), dtype=int32, numpy=
          # array([[ 1,  7],
          #        [13, 19]])>, <tf.Tensor: id=17, shape=(2, 2), dtype=int32, numpy=
          # array([[ 2,  8],
          #        [14, 20]])>, <tf.Tensor: id=18, shape=(2, 2), dtype=int32, numpy=
          # array([[ 3,  9],
          #        [15, 21]])>, <tf.Tensor: id=19, shape=(2, 2), dtype=int32, numpy=
          # array([[ 4, 10],
          #        [16, 22]])>, <tf.Tensor: id=20, shape=(2, 2), dtype=int32, numpy=
          # array([[ 5, 11],
          #        [17, 23]])>]
```

---

# **split**

```python
arr = tf.range(24)
t = tf.reshape(arr, [2, 3, 4])
t0 = tf.unstack(t, axis=2)
t1 = tf.split(t, axis=2, num_or_size_splits=2)
t2 = tf.split(t, axis=2, num_or_size_splits=[1, 2, 1])
print(t) # tf.Tensor(
         # [[[ 0  1  2  3]
         #   [ 4  5  6  7]
         #   [ 8  9 10 11]]
         # 
         #  [[12 13 14 15]
         #   [16 17 18 19]
         #   [20 21 22 23]]], shape=(2, 3, 4), dtype=int32)
print(t0) # [<tf.Tensor: id=6, shape=(2, 3), dtype=int32, numpy=
          # array([[ 0,  4,  8],
          #        [12, 16, 20]])>, <tf.Tensor: id=7, shape=(2, 3), dtype=int32, numpy=
          # array([[ 1,  5,  9],
          #        [13, 17, 21]])>, <tf.Tensor: id=8, shape=(2, 3), dtype=int32, numpy=
          # array([[ 2,  6, 10],
          #        [14, 18, 22]])>, <tf.Tensor: id=9, shape=(2, 3), dtype=int32, numpy=
          # array([[ 3,  7, 11],
          #        [15, 19, 23]])>]
print(t1) # [<tf.Tensor: id=12, shape=(2, 3, 2), dtype=int32, numpy=
          # array([[[ 0,  1],
          #         [ 4,  5],
          #         [ 8,  9]],
          # 
          #        [[12, 13],
          #         [16, 17],
          #         [20, 21]]])>, <tf.Tensor: id=13, shape=(2, 3, 2), dtype=int32, numpy=
          # array([[[ 2,  3],
          #         [ 6,  7],
          #         [10, 11]],
          # 
          #        [[14, 15],
          #         [18, 19],
          #         [22, 23]]])>]
print(t2) # [<tf.Tensor: id=16, shape=(2, 3, 1), dtype=int32, numpy=
          # array([[[ 0],
          #         [ 4],
          #         [ 8]],
          # 
          #        [[12],
          #         [16],
          #         [20]]])>, <tf.Tensor: id=17, shape=(2, 3, 2), dtype=int32, numpy=
          # array([[[ 1,  2],
          #         [ 5,  6],
          #         [ 9, 10]],
          # 
          #        [[13, 14],
          #         [17, 18],
          #         [21, 22]]])>, <tf.Tensor: id=18, shape=(2, 3, 1), dtype=int32, numpy=
          # array([[[ 3],
          #         [ 7],
          #         [11]],
          # 
          #        [[15],
          #         [19],
          #         [23]]])>]
```

---