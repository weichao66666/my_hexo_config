---
layout: article
title: TensorFlow 索引与切片
date: 2019-05-09 09:30:11
tags:
categories: 
copyright: true
---

# **Reference**

* [深度学习与 TensorFlow 2 入门实战](https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469 "https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469")
* [TensorFlow-2.x-Tutorials](https://github.com/dragen1860/TensorFlow-2.x-Tutorials "https://github.com/dragen1860/TensorFlow-2.x-Tutorials")

---

# **Basic indexing**

```python
arr = tf.range(60)
t1 = tf.reshape(arr, [3, 4, 5])
print(t1) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [ 5  6  7  8  9]
          #   [10 11 12 13 14]
          #   [15 16 17 18 19]]
          # 
          #  [[20 21 22 23 24]
          #   [25 26 27 28 29]
          #   [30 31 32 33 34]
          #   [35 36 37 38 39]]
          # 
          #  [[40 41 42 43 44]
          #   [45 46 47 48 49]
          #   [50 51 52 53 54]
          #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t1[0]) # tf.Tensor(
             # [[ 0  1  2  3  4]
             #  [ 5  6  7  8  9]
             #  [10 11 12 13 14]
             #  [15 16 17 18 19]], shape=(4, 5), dtype=int32)
print(t1[0][0]) # tf.Tensor([0 1 2 3 4], shape=(5,), dtype=int32)
print(t1[0][0][0]) # tf.Tensor(0, shape=(), dtype=int32)
```

---

# **numpy-style indexing**

```python
arr = tf.range(60)
t1 = tf.reshape(arr, [3, 4, 5])
print(t1) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [ 5  6  7  8  9]
          #   [10 11 12 13 14]
          #   [15 16 17 18 19]]
          # 
          #  [[20 21 22 23 24]
          #   [25 26 27 28 29]
          #   [30 31 32 33 34]
          #   [35 36 37 38 39]]
          # 
          #  [[40 41 42 43 44]
          #   [45 46 47 48 49]
          #   [50 51 52 53 54]
          #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t1[0]) # tf.Tensor(
             # [[ 0  1  2  3  4]
             #  [ 5  6  7  8  9]
             #  [10 11 12 13 14]
             #  [15 16 17 18 19]], shape=(4, 5), dtype=int32)
print(t1[0, 0]) # tf.Tensor([0 1 2 3 4], shape=(5,), dtype=int32)
print(t1[0, 0, 0]) # tf.Tensor(0, shape=(), dtype=int32)
```

---

# **截取**

```python
t = tf.range(10)
t1 = t[2:]
t2 = t[-2:]
t3 = t[:2]
t4 = t[:-2]
print(t) # tf.Tensor([0 1 2 3 4 5 6 7 8 9], shape=(10,), dtype=int32)
print(t1) # tf.Tensor([2 3 4 5 6 7 8 9], shape=(8,), dtype=int32)
print(t2) # tf.Tensor([8 9], shape=(2,), dtype=int32)
print(t3) # tf.Tensor([0 1], shape=(2,), dtype=int32)
print(t4) # tf.Tensor([0 1 2 3 4 5 6 7], shape=(8,), dtype=int32)
```

---

# **indexing by :**

```python
arr = tf.range(60)
t = tf.reshape(arr, [3, 4, 5])
t1 = t[0, :, :]
t2 = t[:, 0, :]
t3 = t[:, :, 0]
print(t) # tf.Tensor(
         # [[[ 0  1  2  3  4]
         #   [ 5  6  7  8  9]
         #   [10 11 12 13 14]
         #   [15 16 17 18 19]]
         # 
         #  [[20 21 22 23 24]
         #   [25 26 27 28 29]
         #   [30 31 32 33 34]
         #   [35 36 37 38 39]]
         # 
         #  [[40 41 42 43 44]
         #   [45 46 47 48 49]
         #   [50 51 52 53 54]
         #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t1) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [ 5  6  7  8  9]
          #  [10 11 12 13 14]
          #  [15 16 17 18 19]], shape=(4, 5), dtype=int32)
print(t2) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [20 21 22 23 24]
          #  [40 41 42 43 44]], shape=(3, 5), dtype=int32)
print(t3) # tf.Tensor(
          # [[ 0  5 10 15]
          #  [20 25 30 35]
          #  [40 45 50 55]], shape=(3, 4), dtype=int32)
```

---

# **indexing by ::**

```python
arr = tf.range(60)
t = tf.reshape(arr, [3, 4, 5])
t1 = t[:, ::2, :]
t2 = t[0::2, :, :]
t3 = t[0, ::2, :]
print(t) # tf.Tensor(
         # [[[ 0  1  2  3  4]
         #   [ 5  6  7  8  9]
         #   [10 11 12 13 14]
         #   [15 16 17 18 19]]
         # 
         #  [[20 21 22 23 24]
         #   [25 26 27 28 29]
         #   [30 31 32 33 34]
         #   [35 36 37 38 39]]
         # 
         #  [[40 41 42 43 44]
         #   [45 46 47 48 49]
         #   [50 51 52 53 54]
         #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t1) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [10 11 12 13 14]]
          # 
          #  [[20 21 22 23 24]
          #   [30 31 32 33 34]]
          # 
          #  [[40 41 42 43 44]
          #   [50 51 52 53 54]]], shape=(3, 2, 5), dtype=int32)
print(t2) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [ 5  6  7  8  9]
          #   [10 11 12 13 14]
          #   [15 16 17 18 19]]
          # 
          #  [[40 41 42 43 44]
          #   [45 46 47 48 49]
          #   [50 51 52 53 54]
          #   [55 56 57 58 59]]], shape=(2, 4, 5), dtype=int32)
print(t3) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [10 11 12 13 14]], shape=(2, 5), dtype=int32)
```

---

# **...**

```python
arr = tf.range(60)
t = tf.reshape(arr, [3, 4, 5])
t1 = t[0, ...]
t2 = t[..., 0]
t3 = t[0, 0, ...]
t4 = t[..., 0, 0]
print(t) # tf.Tensor(
         # [[[ 0  1  2  3  4]
         #   [ 5  6  7  8  9]
         #   [10 11 12 13 14]
         #   [15 16 17 18 19]]
         # 
         #  [[20 21 22 23 24]
         #   [25 26 27 28 29]
         #   [30 31 32 33 34]
         #   [35 36 37 38 39]]
         # 
         #  [[40 41 42 43 44]
         #   [45 46 47 48 49]
         #   [50 51 52 53 54]
         #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t1) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [ 5  6  7  8  9]
          #  [10 11 12 13 14]
          #  [15 16 17 18 19]], shape=(4, 5), dtype=int32)
print(t2) # tf.Tensor(
          # [[ 0  5 10 15]
          #  [20 25 30 35]
          #  [40 45 50 55]], shape=(3, 4), dtype=int32)
print(t3) # tf.Tensor([0 1 2 3 4], shape=(5,), dtype=int32)
print(t4) # tf.Tensor([ 0 20 40], shape=(3,), dtype=int32)
```

---

# **gather**

```python
arr = tf.range(60)
t = tf.reshape(arr, [3, 4, 5])
t1 = tf.gather(t, axis=0, indices=[0, 2])
t2 = tf.gather(t, axis=0, indices=[0, 2, 1])
t3 = tf.gather(t, axis=1, indices=[0, 2, 1])
t4 = tf.gather(t, axis=1, indices=[0, 2, 1, 3])
t5 = tf.gather(t, axis=2, indices=[0, 2, 1, 4])
t6 = tf.gather(t, axis=2, indices=[0, 2, 1, 4, 3])
print(t1) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [ 5  6  7  8  9]
          #   [10 11 12 13 14]
          #   [15 16 17 18 19]]
          # 
          #  [[40 41 42 43 44]
          #   [45 46 47 48 49]
          #   [50 51 52 53 54]
          #   [55 56 57 58 59]]], shape=(2, 4, 5), dtype=int32)
print(t2) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [ 5  6  7  8  9]
          #   [10 11 12 13 14]
          #   [15 16 17 18 19]]
          # 
          #  [[40 41 42 43 44]
          #   [45 46 47 48 49]
          #   [50 51 52 53 54]
          #   [55 56 57 58 59]]
          # 
          #  [[20 21 22 23 24]
          #   [25 26 27 28 29]
          #   [30 31 32 33 34]
          #   [35 36 37 38 39]]], shape=(3, 4, 5), dtype=int32)
print(t3) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [10 11 12 13 14]
          #   [ 5  6  7  8  9]]
          # 
          #  [[20 21 22 23 24]
          #   [30 31 32 33 34]
          #   [25 26 27 28 29]]
          # 
          #  [[40 41 42 43 44]
          #   [50 51 52 53 54]
          #   [45 46 47 48 49]]], shape=(3, 3, 5), dtype=int32)
print(t4) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [10 11 12 13 14]
          #   [ 5  6  7  8  9]
          #   [15 16 17 18 19]]
          # 
          #  [[20 21 22 23 24]
          #   [30 31 32 33 34]
          #   [25 26 27 28 29]
          #   [35 36 37 38 39]]
          # 
          #  [[40 41 42 43 44]
          #   [50 51 52 53 54]
          #   [45 46 47 48 49]
          #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t5) # tf.Tensor(
          # [[[ 0  2  1  4]
          #   [ 5  7  6  9]
          #   [10 12 11 14]
          #   [15 17 16 19]]
          # 
          #  [[20 22 21 24]
          #   [25 27 26 29]
          #   [30 32 31 34]
          #   [35 37 36 39]]
          # 
          #  [[40 42 41 44]
          #   [45 47 46 49]
          #   [50 52 51 54]
          #   [55 57 56 59]]], shape=(3, 4, 4), dtype=int32)
print(t6) # tf.Tensor(
          # [[[ 0  2  1  4  3]
          #   [ 5  7  6  9  8]
          #   [10 12 11 14 13]
          #   [15 17 16 19 18]]
          # 
          #  [[20 22 21 24 23]
          #   [25 27 26 29 28]
          #   [30 32 31 34 33]
          #   [35 37 36 39 38]]
          # 
          #  [[40 42 41 44 43]
          #   [45 47 46 49 48]
          #   [50 52 51 54 53]
          #   [55 57 56 59 58]]], shape=(3, 4, 5), dtype=int32)
```

---

# **gather_nd**

```python
arr = tf.range(60)
t = tf.reshape(arr, [3, 4, 5])
t1 = tf.gather_nd(t, [0])
t2 = tf.gather_nd(t, [0, 1])
t3 = tf.gather_nd(t, [0, 1, 2])
t4 = tf.gather_nd(t, [[0, 1, 2]])
t5 = tf.gather_nd(t, [[0, 0], [1, 1]])
t6 = tf.gather_nd(t, [[0, 0], [1, 1], [2, 2]])
t7 = tf.gather_nd(t, [[0, 0, 0], [1, 1, 1], [2, 2, 2]])
t8 = tf.gather_nd(t, [[[0, 0, 0], [1, 1, 1], [2, 2, 2]]])
print(t) # tf.Tensor(
         # [[[ 0  1  2  3  4]
         #   [ 5  6  7  8  9]
         #   [10 11 12 13 14]
         #   [15 16 17 18 19]]
         # 
         #  [[20 21 22 23 24]
         #   [25 26 27 28 29]
         #   [30 31 32 33 34]
         #   [35 36 37 38 39]]
         # 
         #  [[40 41 42 43 44]
         #   [45 46 47 48 49]
         #   [50 51 52 53 54]
         #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t1) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [ 5  6  7  8  9]
          #  [10 11 12 13 14]
          #  [15 16 17 18 19]], shape=(4, 5), dtype=int32)
print(t2) # tf.Tensor([5 6 7 8 9], shape=(5,), dtype=int32)
print(t3) # tf.Tensor(7, shape=(), dtype=int32)
print(t4) # tf.Tensor([7], shape=(1,), dtype=int32)
print(t5) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [25 26 27 28 29]], shape=(2, 5), dtype=int32)
print(t6) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [25 26 27 28 29]
          #  [50 51 52 53 54]], shape=(3, 5), dtype=int32)
print(t7) # tf.Tensor([ 0 26 52], shape=(3,), dtype=int32)
print(t8) # tf.Tensor([[ 0 26 52]], shape=(1, 3), dtype=int32)
```

---

# **boolean_mask**

```python
arr = tf.range(60)
t = tf.reshape(arr, [3, 4, 5])
t1 = tf.boolean_mask(t, mask=[True, False, True])
t2 = tf.boolean_mask(t, mask=[True, False, True, True], axis=1)
t3 = tf.boolean_mask(t, mask=[[True, False, True, True], [True, False, False, True], [False, True, False, True]])
print(t) # tf.Tensor(
         # [[[ 0  1  2  3  4]
         #   [ 5  6  7  8  9]
         #   [10 11 12 13 14]
         #   [15 16 17 18 19]]
         # 
         #  [[20 21 22 23 24]
         #   [25 26 27 28 29]
         #   [30 31 32 33 34]
         #   [35 36 37 38 39]]
         # 
         #  [[40 41 42 43 44]
         #   [45 46 47 48 49]
         #   [50 51 52 53 54]
         #   [55 56 57 58 59]]], shape=(3, 4, 5), dtype=int32)
print(t1) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [ 5  6  7  8  9]
          #   [10 11 12 13 14]
          #   [15 16 17 18 19]]
          # 
          #  [[40 41 42 43 44]
          #   [45 46 47 48 49]
          #   [50 51 52 53 54]
          #   [55 56 57 58 59]]], shape=(2, 4, 5), dtype=int32)
print(t2) # tf.Tensor(
          # [[[ 0  1  2  3  4]
          #   [10 11 12 13 14]
          #   [15 16 17 18 19]]
          # 
          #  [[20 21 22 23 24]
          #   [30 31 32 33 34]
          #   [35 36 37 38 39]]
          # 
          #  [[40 41 42 43 44]
          #   [50 51 52 53 54]
          #   [55 56 57 58 59]]], shape=(3, 3, 5), dtype=int32)
print(t3) # tf.Tensor(
          # [[ 0  1  2  3  4]
          #  [10 11 12 13 14]
          #  [15 16 17 18 19]
          #  [20 21 22 23 24]
          #  [35 36 37 38 39]
          #  [45 46 47 48 49]
          #  [55 56 57 58 59]], shape=(7, 5), dtype=int32)
```

---