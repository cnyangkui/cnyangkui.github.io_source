---
title: 使用神经网络识别手写数字
date: 2018-10-07
tags: [ Machine Learning, Neural Network ]
categories: [ Engineering, Machine Learning ]
---

使用MNIST数据集训练神经网络模型。训练数据由28\*28的手写数字的图像组成，输入层包含784=28\*28个神经元。输入像素是灰度级的，值为0.0表示白色，值为1.0表示黑色，中间数值表示逐渐暗淡的灰色。

{% asset_img  intro.png %}

<!--more-->

## Algorithm

{% asset_img  algorithm.png %}
---
[神经网络快速入门](/2018/10/30/ML-NeuralNetworkPPT/)

## Codes

mnist_loader.py: 加载数据
{% codeblock lang:python %}
import numpy as np
import pickle
import gzip

def load_data():
    f = gzip.open('data/mnist.pkl.gz', 'rb')
    training_data, validation_data, test_data = pickle.load(f, encoding="latin1")
    f.close()
    return (training_data, validation_data, test_data)

def load_data_wrapper():
    tr_d, va_d, te_d = load_data()
    # training_data[0]: x; 1*784
    # training_data[1]: y; 0-9
    training_inputs = [np.reshape(x, (784, 1)) for x in tr_d[0]]
    training_results = [vectorized_result(y) for y in tr_d[1]]
    training_data = zip(training_inputs, training_results)
    validation_inputs = [np.reshape(x, (784, 1)) for x in va_d[0]]
    validation_data = zip(validation_inputs, va_d[1])
    test_inputs = [np.reshape(x, (784, 1)) for x in te_d[0]]
    test_data = zip(test_inputs, te_d[1])
    return (training_data, validation_data, test_data)

def vectorized_result(j):
    v = np.zeros((10, 1))
    v[j] = 1.0
    return v
{% endcodeblock %}


network.py: 算法，包括小批量梯度下降、反向传播算法
{% codeblock lang:python %}
import numpy as np
import random

class Network(object):
    
    def __init__(self, sizes):
        """初始化权重和偏置
        
        :param sizes: 每一层神经元数量，类型为list
        
        weights：权重
        biases：偏置
        """
        
        self.sizes = sizes
        self.num_layers = len(sizes)
        self.weights = np.array([np.random.randn(x, y) for x, y in zip(sizes[1:], sizes[:-1])])
        self.biases = np.array([np.random.randn(y, 1) for y in sizes[1:]])
        
    
    def feedforward(self, a):
        """对一组样本x进行预测，然后输出"""
        
        for w, b in zip(self.weights, self.biases):
            a = sigmoid(np.dot(w, a) + b)
        return a
        
    
    def gradient_descent(self, training_data, epochs, mini_batch_size, alpha, test_data=None):
        """MBGD，运行一个或者几个batch时更新一次
        
        :param training_data: 训练数据，每一个样本包括(x, y)，类型为zip
        :epochs: 迭代次数
        :mini_batch_size：每一个小批量数据的数量
        :alpha: 学习率
        :test_data: 测试数据
        """
        
        training_data = list(training_data)
        n = len(training_data)
        if test_data: 
            test_data = list(test_data)
            n_test = len(list(test_data))
        for i in range(epochs):
            random.shuffle(training_data)
            mini_batches = [training_data[k:k+mini_batch_size] for k in range(0, n, mini_batch_size)]
            for mini_batch in mini_batches:
                init_ws_derivative = np.array([np.zeros(w.shape) for w in self.weights])
                init_bs_derivative = np.array([np.zeros(b.shape) for b in self.biases])
                for x, y in mini_batch:
                    activations, zs = self.forwardprop(x) #前向传播
                    delta = self.cost_deviation(activations[-1], zs[-1], y) #计算最后一层误差
                    ws_derivative, bs_derivative = self.backprop(activations, zs, delta) #反向传播，cost func对w和b求偏导
                    init_ws_derivative = init_ws_derivative + ws_derivative
                    init_bs_derivative = init_bs_derivative + bs_derivative
                self.weights = self.weights - alpha / len(mini_batch) * init_ws_derivative
                self.biases = self.biases - alpha / len(mini_batch) * init_bs_derivative
            if test_data:
                print("Epoch {} : {} / {}".format(i, self.evaluate(test_data), n_test)) #识别准确数量/测试数据集总数量
            else:
                print("Epoch {} complete".format(i))


    def forwardprop(self, x):
        """前向传播"""
        
        activation = x
        activations = [x]
        zs = []
        for w, b in zip(self.weights, self.biases):
            z = np.dot(w, activation) + b
            zs.append(z)
            activation = sigmoid(z)
            activations.append(activation)
        return (activations, zs)


    def cost_deviation(self, output, z, y):
        """计算最后一层误差"""
        
        return (output - y) * sigmoid_derivative(z)
    
    
    def backprop(self, activations, zs, delta):
        """反向传播"""
        
        ws_derivative = np.array([np.zeros(w.shape) for w in self.weights])
        bs_derivative = np.array([np.zeros(b.shape) for b in self.biases])
        ws_derivative[-1] = np.dot(delta, activations[-2].transpose())
        bs_derivative[-1] = delta
        
        for l in range(2, self.num_layers):
            z = zs[-l]
            delta = np.dot((self.weights[-l+1]).transpose(), delta) * sigmoid_derivative(z)
            ws_derivative[-l] = np.dot(delta, activations[-l-1].transpose())
            bs_derivative[-l] = delta
        
        return (ws_derivative, bs_derivative)
     
        
    def evaluate(self, test_data):
        """评估"""
        
        test_results = [(np.argmax(self.feedforward(x)), y) for (x, y) in test_data]
        return sum(int(output == y) for (output, y) in test_results)
    

def sigmoid(z):
    return 1.0 / (1.0 + np.exp(-z))


def sigmoid_derivative(z):
    """sigmoid函数偏导"""
    
    return sigmoid(z) * (1 - sigmoid(z))
{% endcodeblock %}


run.py: 运行，训练一个三层（1个输入层、1个隐藏层、1个输出层）的神经网络模型
{% codeblock lang:python %}
import mnist_loader
import network

if __name__ == '__main__':
    training_data, validation_data, test_data = mnist_loader.load_data_wrapper()
    net = network.Network([784, 30, 10]) #28*28
    net.gradient_descent(training_data, 30, 10, 3.0, test_data=test_data)
{% endcodeblock %}

{% asset_link  MNIST_handwritten.zip MNIST数据集及源码下载 %}