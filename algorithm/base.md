# Base

> With a solid foundation, we can start building our algorithm.  
> 基础不牢，地动山摇

## 神经网络类别

- MLP (Multi-layer Perceptron): 多层感知器
- CNN (Convolutional Neural Network): 卷积神经网络
- RNN (Recurrent Neural Network): 循环神经网络
- TRANSFORMER (Transformer): 自注意力网络

## 经典Paper/Net

- LeNet
- AlexNet 
- DenseNet: DenseNet-121, DenseNet-169, DenseNet-201
- ResNet: ResNet-18, ResNet-34, ResNet-50, ResNet-101, ResNet-152
- GAN: Generative Adversarial Network
- Transformer: Action is all you need

## CV四大任务

- Classification
  - LeNet, AlexNet, ResNet, DenseNet
- Location
- Detection
  - Two Stage: R-CNN, Faster-RCNN
  - One Stage: YOLO, SSD, RetinaNet
- Segmentation

## 算法框架/工具

### Optimizer: 优化器

- Batch gradient descent (BGD)
- Stochastic gradient descent (SGD)
- Mini-batch gradient descent (MGD)
- Adam: Like Adadelta and RMSprop

### Loss Function: 损失函数

- CrossEntropyLoss: 交叉熵损失函数
- MeanQuareError: 均方误差
- BinaryCrossentropy: 交叉熵损失函数，一般二分类用
- CategoricalCrossentropy: 分类交叉熵函数

### TensorFlow: Keras
- add Dense: 模型层，包括输入层，隐藏层，输出层等
- compile: 指定优化器，损失函数
  - optimizer:优化器[^overview-opt]
  - loss: 损失函数: loss
- fit: 训练适配数据
- predict: 预测

`Keras`实践的伪代码:


```python
from keras.models import Sequential
from keras.layers import Dense

model = Sequential()
model.add(Dense(units=64, activation='relu', input_dim=100))
model.add(Dense(units=10, activation='softmax'))
model.compile(loss='categorical_crossentropy',
              optimizer='sgd',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5, batch_size=32)
loss_and_metrics = model.evaluate(x_test, y_test, batch_size=128)

classes = model.predict(x_test, batch_size=128)
```

### PyTorch

PyTorch最佳实践之一的伪代码:

```python
import torch

train_data_loader, test_data_loader = get_data_loader()

class Network():
    self __init__(self, input_size, output_size):
        self.model = nn.Sequential(
            nn.Linear(input_size, 64),
            nn.ReLU(),
            nn.Linear(64, output_size),
            nn.Softmax()
        )

    self forward(self, x):
        return self.model(x)

class model:
    def __init__(self, train_data_loader, test_data_loader):
        self.model = Network()
        self.loss_func = torch.nn.CrossEntropyLoss()
        self.optimizer = torch.optim.SGD(self.model.parameters(), lr=0.01)
        self.train_dataloader = train_data_loader
        self.test_dataloader = test_data_loader
        
    def train():
        self.model.train()
        optimizer = torch.optim.SGD(self.model.parameters(), lr=0.01)
        for x,y in self.train_dataloader:
            pred = self.model(x)
            loss = self.loss_func(pred, y)
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
          
    def evaluate():
        self.model.eval()
        correct = 0
        total = 0
        with torch.no_grad():
            for x,y in self.test_dataloader:
                pred = self.model(x)
                _, pred = torch.max(pred, 1)
                total += y.size(0)
                correct += (pred == y).sum()
        return correct.float()/total.float()

for e in epochs:
    for batch in batches:
        train_data.next_batch(batch)
        model.train(batch)
    model.evaluate(test_data)
```

[^overview-opt]:  An overview of gradient descent optimization algorithms
