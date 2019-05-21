---
layout: article
title: TensorFlow 实战——前向传播（张量）
date: 2019-05-20 16:57:27
tags:
categories: 
copyright: true
---

# **Reference**

* [深度学习与 TensorFlow 2 入门实战](https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469 "https://study.163.com/course/courseMain.htm?courseId=1209092816&share=1&shareId=1143588469")
* [TensorFlow-2.x-Tutorials](https://github.com/dragen1860/TensorFlow-2.x-Tutorials "https://github.com/dragen1860/TensorFlow-2.x-Tutorials")

---

# **coding**

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import datasets
import os

os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'

print('GPU', tf.test.is_gpu_available())

(x, y), _ = datasets.mnist.load_data()
x = tf.convert_to_tensor(x, dtype=tf.float32) / 255.
y = tf.convert_to_tensor(y, dtype=tf.int32)
print(x.shape)  # (60000, 28, 28)
print(y.shape)  # (60000,)

# 使用tf.Variable包装,因为tf.Variable有梯度信息
w1 = tf.Variable(tf.random.truncated_normal([784, 256], stddev=0.1))
b1 = tf.Variable(tf.zeros([256]))
w2 = tf.Variable(tf.random.truncated_normal([256, 128], stddev=0.1))
b2 = tf.Variable(tf.zeros([128]))
w3 = tf.Variable(tf.random.truncated_normal([128, 10], stddev=0.1))
b3 = tf.Variable(tf.zeros([10]))
print(w1.shape, b1.shape)  # (784, 256) (256,)
print(w2.shape, b2.shape)  # (256, 128) (128,)
print(w3.shape, b3.shape)  # (128, 10) (10,)

train_db = tf.data.Dataset.from_tensor_slices((x, y)).batch(128)  # 一次取128张图片
lr = 1e-3
for epoch in range(10):
    for step, (x, y) in enumerate(train_db):
        # (128, 28, 28) => (128, 784)
        x = tf.reshape(x, [-1, 28 * 28])

        with tf.GradientTape() as tape:
            # (128, 784)@(784, 256) + (256,) => (128, 256) + (256,) => (128, 256) + (128, 256) => (128, 256)
            h1 = x @ w1 + b1

            h1 = tf.nn.relu(h1)
            h2 = h1 @ w2 + b2
            h2 = tf.nn.relu(h2)
            out = h2 @ w3 + b3

            y_onehot = tf.one_hot(y, depth=10)
            loss = tf.square(y_onehot - out)
            loss = tf.reduce_mean(loss)

        grads = tape.gradient(loss, [w1, b1, w2, b2, w3, b3])  # 计算梯度
        # w1 = w1 - lr * grads[0]会返回tf.Tensor类型的数据，使用assign_sub包装会原地更新
        w1.assign_sub(lr * grads[0])
        b1.assign_sub(lr * grads[1])
        w2.assign_sub(lr * grads[2])
        b2.assign_sub(lr * grads[3])
        w3.assign_sub(lr * grads[4])
        b3.assign_sub(lr * grads[5])

        print(epoch, step, 'loss: ', float(loss))
```

---