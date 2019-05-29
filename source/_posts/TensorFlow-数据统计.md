---
layout: article
title: TensorFlow 数据统计
date: 2019-05-22 09:43:45
tags:
categories: 
copyright: true
---

# **Reference**

* [深度学习与 TensorFlow 2 入门实战](https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469 "https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469")
* [TensorFlow-2.x-Tutorials](https://github.com/dragen1860/TensorFlow-2.x-Tutorials "https://github.com/dragen1860/TensorFlow-2.x-Tutorials")

---

# **norm**

```python
arr = tf.range(-2, 4)
t = tf.cast(tf.reshape(arr, [2, -1]), tf.float32)

# = square root(|-2|^2+|-1|^2+|0|^2+|1|^2+|2|^2+|3|^2) = sqrt(19) = 4.358899
t10 = tf.sqrt(tf.reduce_sum(tf.square(t)))
t11 = tf.norm(t)
t12 = tf.norm(t, ord=2)

# = |-2|+|-1|+|0|+|1|+|2|+|3| = 9
t21 = tf.norm(t, ord=1)

# = cubic root(|-2|^3+|-1|^3+|0|^3+|1|^3+|2|^3+|3|^3) = curt(45) = 3.5568933
t22 = tf.norm(t, ord=3)

# = [square root(|-2|^2+|1|^2), square root(|-1|^2+|2|^2), square root(|0|^2+|3|^2)] = [sqrt(5), sqrt(5), sqrt(9)] = [2.236068, 2.236068, 3]
t31 = tf.norm(t, axis=0)

# = [square root(|-2|^2+|-1|^2+|0|^2), square root(|1|^2+|2|^2+|3|^2)] = [sqrt(5), sqrt(14)] = [2.236068, 3.7416575]
t32 = tf.norm(t, axis=1)

# = [|-2|+|1|, |-1|+|2|, |0|+|3|] = [3, 3, 3]
t41 = tf.norm(t, ord=1, axis=0)

# = [cubic root(|-2|^3+|1|^3), cubic root(|-1|^3+|2|^3), cubic root(|0|^3+|3|^3)] = [curt(9), curt(9), curt(27)] = [2.080084, 2.080084, 3]
t42 = tf.norm(t, ord=3, axis=0)

print(t) # tf.Tensor(
         # [[-2. -1.  0.]
         #  [ 1.  2.  3.]], shape=(2, 3), dtype=float32)
print(t10) # tf.Tensor(4.358899, shape=(), dtype=float32)
print(t11) # tf.Tensor(4.358899, shape=(), dtype=float32)
print(t12) # tf.Tensor(4.358899, shape=(), dtype=float32)
print(t21) # tf.Tensor(9.0, shape=(), dtype=float32)
print(t22) # tf.Tensor(3.5568933, shape=(), dtype=float32)
print(t31) # tf.Tensor([2.236068 2.236068 3.      ], shape=(3,), dtype=float32)
print(t32) # tf.Tensor([2.236068  3.7416575], shape=(2,), dtype=float32)
print(t41) # tf.Tensor([3. 3. 3.], shape=(3,), dtype=float32)
print(t42) # tf.Tensor([2.080084 2.080084 3.      ], shape=(3,), dtype=float32)
```

---

# **reduce_min、reduce_max、reduce_mean、argmin、argmax、argmax**

```python
arr0 = tf.range(12)
arr1 = tf.random.shuffle(arr0)
arr = tf.reshape(arr1, [3, 4])
t11 = tf.reduce_min(arr)
t12 = tf.reduce_min(arr, axis=1)
t21 = tf.reduce_max(arr)
t22 = tf.reduce_max(arr, axis=1)
t31 = tf.reduce_mean(arr)
t32 = tf.reduce_mean(arr, axis=1)
print(arr) # tf.Tensor(
           # [[ 1  8  0  4]
           #  [10 11  2  9]
           #  [ 6  7  5  3]], shape=(3, 4), dtype=int32)
print(tf.argmin(arr)) # tf.Tensor([0 2 0 2], shape=(4,), dtype=int64)
print(tf.argmax(arr)) # tf.Tensor([1 1 2 1], shape=(4,), dtype=int64)
print(tf.argmax(arr)) # tf.Tensor([1 1 2 1], shape=(4,), dtype=int64)
print(t11) # tf.Tensor(0, shape=(), dtype=int32)
print(t12) # tf.Tensor([0 2 3], shape=(3,), dtype=int32)
print(tf.argmin(arr, axis=1)) # tf.Tensor([2 2 3], shape=(3,), dtype=int64)
print(t21) # tf.Tensor(11, shape=(), dtype=int32)
print(t22) # tf.Tensor([ 8 11  7], shape=(3,), dtype=int32)
print(tf.argmax(arr, axis=1)) # tf.Tensor([1 1 1], shape=(3,), dtype=int64)
print(t31) # tf.Tensor(5, shape=(), dtype=int32)
print(t32) # tf.Tensor([3 8 5], shape=(3,), dtype=int32)
print(tf.argmax(arr, axis=1)) # tf.Tensor([1 1 1], shape=(3,), dtype=int64)
```

---

# **equal**

```python
arr11 = tf.random.uniform([1, 10], dtype=tf.int32, maxval=4)
arr1 = tf.squeeze(arr11)
arr21 = tf.random.uniform([1, 10], dtype=tf.int32, maxval=4)
arr2 = tf.squeeze(arr21)
t = tf.equal(arr1, arr2)
t1 = tf.reduce_sum(tf.cast(t, dtype=tf.int32))
print(arr1) # tf.Tensor([1 0 2 3 3 1 1 1 0 1], shape=(10,), dtype=int32)
print(arr2) # tf.Tensor([3 0 2 2 1 3 1 2 1 2], shape=(10,), dtype=int32)
print(t) # tf.Tensor([False  True  True False False False  True False False False], shape=(10,), dtype=bool)
print(t1) # tf.Tensor(3, shape=(), dtype=int32)
```

---

# **unique**

```python
arr0 = tf.random.uniform([1, 10], dtype=tf.int32, maxval=8)
arr = tf.squeeze(arr0)
y, idx = tf.unique(arr)
arr1 = tf.gather(y, idx)
print(arr) # tf.Tensor([0 1 2 3 3 1 3 7 5 7], shape=(10,), dtype=int32)
print(y) # tf.Tensor([0 1 2 3 7 5], shape=(6,), dtype=int32)
print(idx) # tf.Tensor([0 1 2 3 3 1 3 4 5 4], shape=(10,), dtype=int32)
print(arr1) # tf.Tensor([0 1 2 3 3 1 3 7 5 7], shape=(10,), dtype=int32)
```

---