---
redirect_from: /_posts/2023-4-22-基于pytorch和CIFAR10的简单神经网络模型建立/
title: 基于pytorch和CIFAR10的简单神经网络模型建立
tag:
    -深度学习
---
# 导入所需的库 
```
import torch
from torch import nn
import torchvision.datasets
from torch.utils.tensorboard import SummaryWriter
from torch.utils.data import DataLoader
```
# 定义神经网络模型
可以设计一个由卷积层和全连接层组成的简单神经网络模型。例如，以下示例代码定义了一个包含两个卷积层和三个全连接层的网络。
```
class CK(nn.Module):
    def __init__(self):
        super(CK, self).__init__()
        self.model=nn.Sequential(
            nn.Conv2d(3,32,5,1,2),
            nn.MaxPool2d(2),
            nn.Conv2d(32,32,5,1,2),
            nn.MaxPool2d(2),
            nn.Conv2d(32,64,5,1,2),
            nn.MaxPool2d(2),
            nn.Flatten(),
            nn.Linear(64*4*4,64),
            nn.Linear(64,10)
        )
    def forward(self,x):
        x=self.model(x)
        return x
ck=CK()
```
# 加载数据集
可以使用PyTorch提供的datasets.CIFAR10类加载CIFAR10数据集，同时需要对图像进行预处理。
```
train_data=torchvision.datasets.CIFAR10(root="D:\\pytorch学习",train=True,transform=torchvision.transforms.ToTensor(),
                                        download=True)
 
test_data=torchvision.datasets.CIFAR10(root="D:\\pytorch学习",train=False,transform=torchvision.transforms.ToTensor(),
                                        download=True)
train_dataloader=DataLoader(train_data,batch_size=64)
test_dataloader=DataLoader(test_data,batch_size=64)
```
# 定义损失函数和优化器
此处使用SGD
```
#创建损失函数
loss_fn=nn.CrossEntropyLoss()
 
#优化器
optimizer = torch.optim.SGD(ck.parameters(),lr=0.01)
```
# 训练网络模型 
在训练过程中，遍历数据集中的每个图像，并将其输入神经网络中。然后计算网络输出与真实标签之间的误差，并使用梯度下降法更新网络参数，从而使误差逐步减小并学习到更好的特征。在tensorboard上绘制出相关曲线并保存模型参数。

```
#设置训练网络的一些参数
#记录训练次数
total_train_step = 0
#记录测试次数
total_test_step = 1
#训练轮数
epoch = 30
 
#添加tensorboard
writer=SummaryWriter("logs_train")
 
for i in range(epoch):
    print("********第{}轮训练开始********".format(i+1))
 
    # 动态设置随机种子
    torch.manual_seed(i)
 
    #训练步骤开始
    ck.train()
    for data in train_dataloader:
        imgs,targets=data
        outputs=ck(imgs)
        loss = loss_fn(outputs,targets)
 
        #优化器模型
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
 
        total_train_step = total_train_step + 1
        if total_train_step % 100==0:
            print("训练次数：{},loss:{}".format(total_train_step,loss.item()))
            writer.add_scalar("train_loss",loss.item(),total_train_step)
 
    #测试步骤开始
    ck.eval()
    total_test_loss=0
    total_accuracy = 0
    with torch.no_grad():
        for data in test_dataloader:
            imgs, targets =data
            outputs = ck(imgs)
            loss = loss_fn(outputs,targets)
            total_test_loss=total_test_loss + loss.item()
            accuracy=(outputs.argmax(1) == targets).sum()
            total_accuracy = total_accuracy + accuracy
        print("整体测试集上的Loss:{}".format(total_test_loss))
        print("整体测试集上的正确率:{}".format(total_accuracy/test_data_size))
        writer.add_scalar("test_accuracy", total_accuracy/test_data_size,total_test_step)
        total_accuracy = 0
        writer.add_scalar("test_loss",total_test_loss,total_test_step)
        total_test_step += 1
        torch.save(ck.state_dict(),"ck_{}.pth".format(i))
        print("模型已保存")
 
writer.close()
```
