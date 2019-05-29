---
layout: article
title: TensorFlow 张量排序
date: 2019-05-28 09:45:13
tags:
categories: 
copyright: true
---

# **Reference**

* [深度学习与 TensorFlow 2 入门实战](https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469 "https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469")
* [TensorFlow-2.x-Tutorials](https://github.com/dragen1860/TensorFlow-2.x-Tutorials "https://github.com/dragen1860/TensorFlow-2.x-Tutorials")

---

# **sort、argsort**

```python
arr = tf.range(10)
arr1 = tf.random.shuffle(arr)
arr21 = tf.sort(arr1, direction='DESCENDING')
arr22 = tf.argsort(arr1, direction='DESCENDING')
arr31 = tf.sort(arr1, direction='ASCENDING')
arr32 = tf.argsort(arr1, direction='ASCENDING')
print(arr1) # tf.Tensor([8 5 3 2 4 0 7 1 9 6], shape=(10,), dtype=int32)
print(arr21) # tf.Tensor([9 8 7 6 5 4 3 2 1 0], shape=(10,), dtype=int32)
print(arr22) # tf.Tensor([8 0 6 9 1 4 2 3 7 5], shape=(10,), dtype=int32)
print(arr31) # tf.Tensor([0 1 2 3 4 5 6 7 8 9], shape=(10,), dtype=int32)
print(arr32) # tf.Tensor([5 7 3 2 4 1 9 6 0 8], shape=(10,), dtype=int32)
```

---

# **top_k**

```python
arr = tf.range(10)
arr1 = tf.random.shuffle(arr)
res = tf.math.top_k(arr1, 2)
print(arr1) # tf.Tensor([8 7 3 2 9 5 0 6 4 1], shape=(10,), dtype=int32)
print(res.values) # tf.Tensor([9 8], shape=(2,), dtype=int32)
print(res.indices) # tf.Tensor([4 0], shape=(2,), dtype=int32)
```

## **计算 top_k 概率**

```python
def accuracy(output, target, topk=(1,)):
    maxk = max(topk)
    batch_size = target.shape[0]

    pred = tf.math.top_k(output, maxk).indices
    print(pred) # tf.Tensor(
                # [[1 0 2]
                #  [0 1 2]
                #  [1 0 2]
                #  [1 0 2]], shape=(4, 3), dtype=int32)

    # 转置
    pred = tf.transpose(pred, perm=[1, 0])
    print(pred) # tf.Tensor(
                # [[1 0 1 1]
                #  [0 1 0 0]
                #  [2 2 2 2]], shape=(3, 4), dtype=int32)

    target_ = tf.broadcast_to(target, pred.shape)
    print(target_) # tf.Tensor(
                   # [[0 1 0 2]
                   #  [0 1 0 2]
                   #  [0 1 0 2]], shape=(3, 4), dtype=int32)
    correct = tf.equal(pred, target_)
    print(correct) # tf.Tensor(
                   # [[False False False False]
                   #  [ True  True  True False]
                   #  [False False False  True]], shape=(3, 4), dtype=bool)

    res = []
    for k in topk:
        correct_k = tf.cast(tf.reshape(correct[:k], [-1]), dtype=tf.float32)
        correct_k = tf.reduce_sum(correct_k)
        acc = float(correct_k * (100.0 / batch_size))
        res.append(acc)

    return res


output = tf.random.normal([4, 3])
output = tf.math.softmax(output, axis=1)
print('output:', output.numpy()) # output: [[0.16469155 0.7675324  0.06777601]
                                 #  [0.50406045 0.31256208 0.18337746]
                                 #  [0.24097656 0.5227547  0.23626871]
                                 #  [0.2440812  0.5844257  0.17149314]]
target = tf.random.uniform([4], maxval=3, dtype=tf.int32)
print('target:', target.numpy()) # target: [0 1 0 2]
acc = accuracy(output, target, topk=(1, 2, 3))
print('top1-6 acc:', acc) # top1-6 acc: [0.0, 75.0, 100.0]
```

---