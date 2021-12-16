# base network

## 算法模型

### Keras
- add Dense: 模型层，包括输入层，隐藏层，输出层等
- compile: 指定优化器，损失函数
  - optimizer:优化器[^overview-opt]
    - Batch gradient descent (BGD)
    - Stochastic gradient descent (SGD)
    - Mini-batch gradient descent (MGD)
    - Adam: Like Adadelta and RMSprop
  - loss: 损失函数: loss
    - Mean-square error (MSE)
    - binary_crossentropy: 交叉熵损失函数，一般二分类
    - categorical_crossentropy: 分类交叉熵函数
- fix: 训练适配数据
- predict: 预测

## 经典Paper

- LeNet 模型
- GAN 论文
- AlexNet

[^overview-opt]:  An overview of gradient descent optimization algorithms
