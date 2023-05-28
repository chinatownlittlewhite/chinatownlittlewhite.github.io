---
redirect_from: /_post/2023-3-12-matlab实现组合差异进化算法(CoDE)(基础简化版)/
title: matlab实现组合差异进化算法(CoDE)(基础简化版)
tag:
    -优化算法
---
# 概述 
差分进化算法(Differential Evolution，DE)由Storn和Price于1995年首次提出。经过对大量的试验向量产生策略和控制参数设置进行大量的研究，我们已经知道哪些实验向量产生策略适合于全局搜索，哪些试验向量产生策略可用于求解具有旋转特性的问题，哪些控制参数设置可以加速收敛，哪些控制对可分离问题有效，这些实验结论对改进DE的性能非常有帮助，但是在设置DE的时候却没有很好的利用。  
沿着以上的思路，组合差异进化算法(CoDE)被提出。 
# 算法流程 
相比于普通的DE算法，CoDE采用了三组不同的控制参数F与CR值
![](https://img-blog.csdnimg.cn/img_convert/76948dc39ed12c85a2b68f6d618076d8.png) 

这第一组用于求解可分离问题，第二组值用于用于维持种群多样性，第三组值取较大的F值，使种群在较大的搜索空间进行开采，以加快收敛速度。

# 伪代码描述
![](https://img-blog.csdnimg.cn/img_convert/f993fe62d747a9f7cba12b16506ddc4e.png) 
所选取的三个试验向量的产生策略为 
![](https://img-blog.csdnimg.cn/img_convert/5cf7b8d2c652ec1df889bb51f2a39fb7.png) 
注意：current-to-rand/1算子没有使用二项式交叉 
# 代码实现 
下面附上我的代码实现(初学者，代码可能有许多可以改进的地方，有大佬请帮忙指正)
```
close all
clear all
clc
max_epoch=2000;     %最大迭代次数
F=[1.0 1.0 0.8];
CR=[0.1 0.9 0.2];
NP=30;        %种群数量
D=30;         %染色体长度
a=-100;        %上下限
b=100;
v1=zeros(NP,D);        
v2=zeros(NP,D);
v3=zeros(NP,D);
x=rand(NP,D)*(b-a)+a;   %初始化初始种群
for i=1:1:NP        
    ob(i)=sum(x(i,:).^2);  %求初始种群的适应度
end        
trace(1)=min(ob);
for i=1:max_epoch        %进行迭代
    num = randperm(3,1);        
    for m=1:1:NP
        r1=randi([1,NP],1,1);
        while (r1==m)
            r1=randi([1,NP],1,1);
        end
        r2=randi([1,NP],1,1);
        while(r1==r2)||(r2==m)
            r2=randi([1,NP],1,1);
        end
        r3=randi([1,NP],1,1);
        while(r3==m)||(r3==r2)||(r3==r1)
            r3=randi([1,NP],1,1);
        end
        v1(m,:)=x(r1,:)+F(:,num).*(x(r2,:)-x(r3,:)); %生成变异体
    end
    for m=1:1:NP           %修补染色体(处理越界数据)
        for n=1:1:D
            if v1(m,n)<=a
                v1(m,n)=a;
            end
            if v1(m,n)>=b
                v1(m,n)=b;
            end
        end
    end
    for c=1:size(v1,1)        %交叉操作
        for k=1:size(v1,2)
            if rand<CR(num)
                v1(c,k)=x(c,k);
            end
        end
    end
    num = randperm(3,1);
    for m=1:1:NP
        r1=randi([1,NP],1,1);
        while (r1==m)
            r1=randi([1,NP],1,1);
        end
        r2=randi([1,NP],1,1);
        while(r1==r2)||(r2==m)
            r2=randi([1,NP],1,1);
        end
        r3=randi([1,NP],1,1);
        while(r3==m)||(r3==r2)||(r3==r1)
            r3=randi([1,NP],1,1);
        end
        r4=randi([1,NP],1,1);
        while(r1==r4)||(r4==m)||(r4==r3)||(r4==r2)
            r4=randi([1,NP],1,1);
        end
        r5=randi([1,NP],1,1);
        while(r1==r5)||(r5==m)||(r5==r3)||(r5==r2)||(r4==r5)
            r5=randi([1,NP],1,1);
        end
        v2(m,:)=x(r1,:)+F(1,num)*(x(r2,:)-x(r3,:))+F(1,num)*(x(r4,:)-x(r5,:)); 
    end
    for m=1:1:NP        
        for n=1:1:D
            if v2(m,n)<=a
                v2(m,n)=a;
            end
            if v2(m,n)>=b
                v2(m,n)=b;
            end
        end
    end
    for c=1:size(v2,1)
        for k=1:size(v2,2)
            if rand<CR(num)
                v2(c,k)=x(c,k);
            end
        end
    end
    num = randperm(3,1);
    for m=1:1:NP
        r1=randi([1,NP],1,1);
        while (r1==m)
            r1=randi([1,NP],1,1);
        end
        r2=randi([1,NP],1,1);
        while(r1==r2)||(r2==m)
            r2=randi([1,NP],1,1);
        end
        r3=randi([1,NP],1,1);
        while(r3==m)||(r3==r2)||(r3==r1)
            r3=randi([1,NP],1,1);
        end
        v3(m,:)=x(m,:)+F(1,num)*(x(r1,:)-x(r2,:))+F(1,num)*(x(r2,:)-x(r3,:)); 
    end
    for m=1:1:NP       
        for n=1:1:D
            if v3(m,n)<=a
                v3(m,n)=a;
            end
            if v3(m,n)>=b
                v3(m,n)=b;
            end
        end
    end
    for c=1:size(v3,1)
        for k=1:size(v3,2)
            if rand<CR(num)
                v1(c,k)=x(c,k);
            end
        end
    end
    for k=1:1:NP        
    ob1(k)=sum(v1(k,:).^2);  %重新测试适应度
    end        
     for k=1:1:NP        
    ob2(k)=sum(v2(k,:).^2);  
    end        
    for k=1:1:NP        
    ob3(k)=sum(v3(k,:).^2);  
    end
    for m=1:NP        %优胜劣汰，得到新种群
        if ob1(m)<ob(m)    
            x(m,:)=v1(m,:);
        end
    end
    for m=1:NP
        if ob2(m)<ob(m)    
            x(m,:)=v2(m,:);
        end
    end
    for m=1:NP
        if ob3(m)<ob(m)    
            x(m,:)=v3(m,:);
        end
    end
    for m=1:NP
        ob(m)=sum(x(m,:).^2);
    end
    trace(i+1)=min(ob);
    tt=min(ob);
end
x(1,:);
figure(1);
plot(trace);
title(['组合差分进化算法(CoDE)', '最小值: ', num2str(tt)]);
xlabel('迭代次数'); 
ylabel('目标函数值');
```
# 运行结果
![](https://img-blog.csdnimg.cn/img_convert/88ddd05f4c9bb7e4a8c4fda3031ac5c4.png) 


### 参考文献引用自中南大学王勇的博士学位论文——基于进化算法求解复杂连续优化问题的研究。
