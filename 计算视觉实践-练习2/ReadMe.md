

- [CNN Explainer 试用报告](#cnn-explainer-试用报告)
    - [**卷积层**](#卷积层)
    - [**激活层**](#激活层)
    - [**池化层**](#池化层)
    - [**Flatten Layer**](#flatten-layer)
- [LeNet-5 在 MNIST 数据集的训练和测试](#lenet-5-在-mnist-数据集的训练和测试)
    - [**定义超参数**](#定义超参数)
    - [**构建transform**](#构建transform)
    - [**下载并加载数据集**](#下载并加载数据集)
    - [**构建网络模型和优化器**](#构建网络模型和优化器)
    - [**训练方法**](#训练方法)
    - [**测试方法**](#测试方法)
    - [**训练并测试**](#训练并测试)



### CNN Explainer 试用报告

​	本试用报告是基于 [CNN Explainer 官方网站]( [poloclub/cnn-explainer: Learning Convolutional Neural Networks with Interactive Visualization. (github.com)](https://github.com/poloclub/cnn-explainer?tab=readme-ov-file)) 提供的 [demo]([CNN Explainer (poloclub.github.io)](https://poloclub.github.io/cnn-explainer/))。

​	CNN Explainer 把Tiny VGG 神经网络中的每层输出都可视化，并且提供了交互操作，可以更详细看到每层网络的输入和对应的输出。

![image-20240503143009568](assets/image-20240503143009568-1714717891104-2.png)

随着CNN网络层数增加，网络更能捕捉到输入图片的特征。

![image-20240503143802802](assets/image-20240503143802802-1714718285444-1-1714718288327-3.png)

我们还可以从CNN Explainer了解到 CNN 网络各组件的作用：

##### **卷积层**

卷积层是 CNN 的基础，因为它们包含了学习到的核（权重），这些核可以提取出区分不同图像的特征，而这正是我们需要的分类功能！当你与卷积层交互时，你会注意到前几层与卷积层之间的链接。每个链接都代表一个独特的内核，用于卷积操作，生成当前卷积神经元的输出或激活图。

##### **激活层**

神经网络在现代技术中极为普遍--因为它们非常精确！当今性能最高的 CNN 由数量惊人的层组成，能够学习越来越多的特征。这些开创性的 CNN 能够达到如此高的精确度，部分原因在于它们的非线性。ReLU 在模型中应用了亟需的非线性。非线性是产生非线性决策边界的必要条件，因此输出不能写成输入的线性组合。如果没有非线性激活函数，深度 CNN 架构就会退化为单一的等价卷积层，其性能也不会太好。之所以特别使用 ReLU 激活函数作为非线性激活函数，而不是 Sigmoid 等其他非线性函数，是因为根据经验观察，使用 ReLU 的 CNN 比同类产品的训练速度更快。

##### **池化层** 

在不同的 CNN 架构中，池化层有多种类型，但它们的目的都是逐渐缩小网络的空间范围，从而减少网络的参数和整体计算量。最大池化运算要求在架构设计时选择内核大小和步长。选定后，该操作会在输入上滑动具有指定步长的内核，同时只从输入中选择每个内核切片上的最大值，从而得出输出值。点击上面网络中的池化神经元，即可查看这一过程。

##### **Flatten Layer**

这一层将网络中的三维层转换为一维向量，以适应全连接层的分类输入。例如，一个 5x5x2 的张量将被转换成一个大小为 50 的向量。网络的前几个卷积层从输入图像中提取了特征，现在是对特征进行分类的时候了。我们使用 softmax 函数对这些特征进行分类，这需要一维输入。这就是为什么需要扁平化层。

### LeNet-5 在 MNIST 数据集的训练和测试

##### **定义超参数**

```python
# --------------step1: 定义超参数-------------------------
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")  # 是否用GPU
EPOCHS = 10  # 数据集的训练次数
BATCH_SIZE = 16  # 每批处理的数据 16/32/64/128
```

##### **构建transform**

```python
# -------------step2: 构建transform（对图像做处理）---------
transform = transforms.Compose([
    transforms.ToTensor(),  # 将图片转成成tensor
    transforms.Normalize((0.1307, ), (0.3081, ))  # 标准化 => x' = （x-μ）/σ
])
```

##### **下载并加载数据集**

```python
# -------------step3: 下载并加载数据集------------------
# 下载数据集
train_set = datasets.MNIST("data_sets", train=True, download=True, transform=transform)
test_set = datasets.MNIST("data_sets", train=False, download=True, transform=transform)
# 加载数据集
train_loader = DataLoader(train_set, batch_size=BATCH_SIZE, shuffle=True)
test_loader = DataLoader(test_set, batch_size=BATCH_SIZE, shuffle=True)
```

##### **构建网络模型和优化器**

```python
# -------------step4: 构建网络模型--------------------
class LeNet(nn.Module):
    """
    A neural network with:
      2 Conbolutions
      3 Full connnection
    """
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 6, (5, 5), (1, ), 2)  # 1:输入通道, 6:输出通道, (5,5):kernel_size, 1:stride, 2:padding
        self.conv2 = nn.Conv2d(6, 16, (5, 5))  # 6:输入通道, 16:输出通道, (5,5):kernel_size
        self.fc1 = nn.Linear(16*5*5, 120)   # 16*5*5:输入通道, 120:输出通道
        self.fc2 = nn.Linear(120, 84)  # 输入通道:120, 输出通道:84
        self.fc3 = nn.Linear(84, 10)  # 输入通道:84, 输出通道:10

    def forward(self, x):
        x = self.conv1(x)  # 输入：batch*1*28*28 输出：batch*6*28*28 （28+2*2-5+1=28）
        x = F.relu(x)      # 激活层 输出：batch**6*28*28
        x = F.max_pool2d(x, 2, 2)  # 池化层/下采样 输入：batch*6*28*28 输出：batch*6*14*14

        x = self.conv2(x)  # 输入：batch*6*14*14 输出：batch*16*10*10  （14+2*0-5+1=10）
        x = F.relu(x)  # 激活层 输出：batch*16*10*10
        x = F.max_pool2d(x, 2, 2)  # 池化层/下采样 输入：batch*16*10*10 输出：16*5*5

        x = x.view(x.size(0), -1)  # 拉平 （-1指自动计算维度） 16*5*5

        x = self.fc1(x)  # 输入：batch*16*5*5  输出： batch*120
        x = F.relu(x)    # 激活层 输出：batch*120
        x = self.fc2(x)  # 输入：batch*120  输出：batch*84
        x = F.relu(x)    # 激活层  输出：batch*84
        x = self.fc3(x)  # 输入：batch*84  输出：batch*10
        output = F.softmax(x, dim=1)  # 计算分类后每个数字的概率值

        return output
    
    # ----------------step:5 定义优化器--------------------------
model = LeNet().to(DEVICE)
optimizer = optim.Adam(model.parameters())
```

##### **训练方法**

```python
# ----------------step6: 定义训练方法-----------------------
def train_model(my_model, device, trains_loader, optimizers, epoches):
    # 模型训练
    my_model.train()
    for batch_idx, (data, target) in enumerate(trains_loader):
        # 将data和target部署到DEVICE上去
        data, target = data.to(device), target.to(device)
        # 将梯度初始化为0
        optimizers.zero_grad()
        # 训练所得的结果
        output = my_model(data)
        # 计算交叉熵损失
        loss = F.cross_entropy(output, target)
        # 反向传播
        loss.backward()
        # 更新参数
        optimizers.step()
        # 每100batch_size打印一次log
        if batch_idx % 1000 == 0:
            print("Training Epoch:{} \t Loss:{:.5f}".format(epoches, loss.item()))
```

##### **测试方法** 

```python
# ----------------step7: 定义测试方法------------------------
def test_model(my_model, device, test_loder):
    my_model.eval()  # 模型验证
    correct = 0.0    # 正确率
    test_loss = 0.0   # 测试损失
    with torch.no_grad():  # 测试时不计算梯度，也不进行反向传播
        for data, target in test_loder:
            # 将data和target部署到device上
            data, target = data.to(device), target.to(device)
            # 测试所得的结果
            output = my_model(data)
            # 计算交叉熵损失
            test_loss += F.cross_entropy(output, target).item()
            # 找到概率最大的下标
            predict = output.argmax(dim=1)
            # predict = torch.max(output, dim=1)
            correct += predict.eq(target.view_as(predict)).sum().item()  # 累计正确的值
        # 计算平均损失
        avg_loss = test_loss / len(test_loder.dataset)
        # 计算准确率
        correct_ratio = 100 * correct / len(test_loder.dataset)
        print("Average_loss in test:{:.5f}\t Accuracy:{:.5f}\n".format(
            avg_loss, correct_ratio
        ))

```

##### **训练并测试**

```python
# --------------step8: 训练模型----------------------------
for epoch in range(1, EPOCHS+1):
    train_model(model, DEVICE, train_loader, optimizer, epoch)
    test_model(model, DEVICE, test_loader)
```

