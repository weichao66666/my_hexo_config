---
layout: article
title: TensorFlow broadcasting
date: 2019-05-14 09:33:53
tags:
categories: 
copyright: true
---

# **Reference**

* [深度学习与 TensorFlow 2 入门实战](https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469 "https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469")
* [TensorFlow-2.x-Tutorials](https://github.com/dragen1860/TensorFlow-2.x-Tutorials "https://github.com/dragen1860/TensorFlow-2.x-Tutorials")

---

# **broadcasting**

```python
arr = tf.range(60)
arr1 = tf.reshape(arr, [3, 4, 5])
t = tf.cast(arr1, dtype=tf.float32)
t1 = tf.ones(5)
t2 = tf.ones([4, 1])
t3 = tf.ones([3, 4, 1])
t11 = t + t1
t21 = t + t2
t31 = t + t3
print(t) # tf.Tensor(
[[[ 0.  1.  2.  3.  4.]
  [ 5.  6.  7.  8.  9.]
  [10. 11. 12. 13. 14.]
  [15. 16. 17. 18. 19.]]

 [[20. 21. 22. 23. 24.]
  [25. 26. 27. 28. 29.]
  [30. 31. 32. 33. 34.]
  [35. 36. 37. 38. 39.]]

 [[40. 41. 42. 43. 44.]
  [45. 46. 47. 48. 49.]
  [50. 51. 52. 53. 54.]
  [55. 56. 57. 58. 59.]]], shape=(3, 4, 5), dtype=float32)
print(t1) # tf.Tensor([1. 1. 1. 1. 1.], shape=(5,), dtype=float32)
print(t2) # tf.Tensor(
          # [[1.]
          #  [1.]
          #  [1.]
          #  [1.]], shape=(4, 1), dtype=float32)
print(t3) # tf.Tensor(
          # [[[1.]
          #   [1.]
          #   [1.]
          #   [1.]]
          # 
          #  [[1.]
          #   [1.]
          #   [1.]
          #   [1.]]
          # 
          #  [[1.]
          #   [1.]
          #   [1.]
          #   [1.]]], shape=(3, 4, 1), dtype=float32)
print(t11) # tf.Tensor(
           # [[[ 1.  2.  3.  4.  5.]
           #   [ 6.  7.  8.  9. 10.]
           #   [11. 12. 13. 14. 15.]
           #   [16. 17. 18. 19. 20.]]
           # 
           #  [[21. 22. 23. 24. 25.]
           #   [26. 27. 28. 29. 30.]
           #   [31. 32. 33. 34. 35.]
           #   [36. 37. 38. 39. 40.]]
           # 
           #  [[41. 42. 43. 44. 45.]
           #   [46. 47. 48. 49. 50.]
           #   [51. 52. 53. 54. 55.]
           #   [56. 57. 58. 59. 60.]]], shape=(3, 4, 5), dtype=float32)
print(t21) # tf.Tensor(
           # [[[ 1.  2.  3.  4.  5.]
           #   [ 6.  7.  8.  9. 10.]
           #   [11. 12. 13. 14. 15.]
           #   [16. 17. 18. 19. 20.]]
           # 
           #  [[21. 22. 23. 24. 25.]
           #   [26. 27. 28. 29. 30.]
           #   [31. 32. 33. 34. 35.]
           #   [36. 37. 38. 39. 40.]]
           # 
           #  [[41. 42. 43. 44. 45.]
           #   [46. 47. 48. 49. 50.]
           #   [51. 52. 53. 54. 55.]
           #   [56. 57. 58. 59. 60.]]], shape=(3, 4, 5), dtype=float32)
print(t31) # tf.Tensor(
           # [[[ 1.  2.  3.  4.  5.]
           #   [ 6.  7.  8.  9. 10.]
           #   [11. 12. 13. 14. 15.]
           #   [16. 17. 18. 19. 20.]]
           # 
           #  [[21. 22. 23. 24. 25.]
           #   [26. 27. 28. 29. 30.]
           #   [31. 32. 33. 34. 35.]
           #   [36. 37. 38. 39. 40.]]
           # 
           #  [[41. 42. 43. 44. 45.]
           #   [46. 47. 48. 49. 50.]
           #   [51. 52. 53. 54. 55.]
           #   [56. 57. 58. 59. 60.]]], shape=(3, 4, 5), dtype=float32)

t4 = tf.ones([3, 4, 2])
t41 = t + t4 # tensorflow.python.framework.errors_impl.InvalidArgumentError: Incompatible shapes: [3,4,5] vs. [3,4,2] [Op:Add] name: add/
```

---

# **broadcast_to**

```python
arr = tf.range(12)
arr1 = tf.reshape(arr, [3, 4, 1])
t = tf.cast(arr1, dtype=tf.float32)
t1 = tf.broadcast_to(t, [3, 4, 5])
print(t) # tf.Tensor(
         # [[[ 0.]
         #   [ 1.]
         #   [ 2.]
         #   [ 3.]]
         # 
         #  [[ 4.]
         #   [ 5.]
         #   [ 6.]
         #   [ 7.]]
         # 
         #  [[ 8.]
         #   [ 9.]
         #   [10.]
         #   [11.]]], shape=(3, 4, 1), dtype=float32)
print(t1) # tf.Tensor(
          # [[[ 0.  0.  0.  0.  0.]
          #   [ 1.  1.  1.  1.  1.]
          #   [ 2.  2.  2.  2.  2.]
          #   [ 3.  3.  3.  3.  3.]]
          # 
          #  [[ 4.  4.  4.  4.  4.]
          #   [ 5.  5.  5.  5.  5.]
          #   [ 6.  6.  6.  6.  6.]
          #   [ 7.  7.  7.  7.  7.]]
          # 
          #  [[ 8.  8.  8.  8.  8.]
          #   [ 9.  9.  9.  9.  9.]
          #   [10. 10. 10. 10. 10.]
          #   [11. 11. 11. 11. 11.]]], shape=(3, 4, 5), dtype=float32)
```

---

# **tile**

```python
a = tf.ones([3, 4])
a1 = tf.broadcast_to(a, [2, 3, 4])
a2 = tf.expand_dims(a, axis=0)
a21 = tf.tile(a2, [2, 1, 1])
print(a) # tf.Tensor(
         # [[1. 1. 1. 1.]
         #  [1. 1. 1. 1.]
         #  [1. 1. 1. 1.]], shape=(3, 4), dtype=float32)
print(a1) # tf.Tensor(
          # [[[1. 1. 1. 1.]
          #   [1. 1. 1. 1.]
          #   [1. 1. 1. 1.]]
          # 
          #  [[1. 1. 1. 1.]
          #   [1. 1. 1. 1.]
          #   [1. 1. 1. 1.]]], shape=(2, 3, 4), dtype=float32)
print(a2) # tf.Tensor(
          # [[[1. 1. 1. 1.]
          #   [1. 1. 1. 1.]
          #   [1. 1. 1. 1.]]], shape=(1, 3, 4), dtype=float32)
print(a21) # tf.Tensor(
           # [[[1. 1. 1. 1.]
           #   [1. 1. 1. 1.]
           #   [1. 1. 1. 1.]]
           # 
           #  [[1. 1. 1. 1.]
           #   [1. 1. 1. 1.]
           #   [1. 1. 1. 1.]]], shape=(2, 3, 4), dtype=float32)
```

---