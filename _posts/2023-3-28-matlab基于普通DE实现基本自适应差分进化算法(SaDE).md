---
title:matlab基于普通DE实现基本自适应差分进化算法(SaDE)
tag:
  -优化算法
---
# 提出背景 
由于给定所求的函数条件下，我们无法准确知道求解该函数适于怎样取值的变异算子F和交叉概率CR，由此推出自适应差分进化算法(DE)。 
# 实现方法 
为了使所用的F与CR尽量符合所求函数的要求，我们设定一个初始的交叉概率最大值CEm，初始值设为0.5，并生成服从正态分布且取值在(0.3，0.5)区间的随机数F。设定一个自适应取值的节点LP，即在迭代次数为LP是进行自适应值的改变。为了结合各个变异策略的优点，我们再次选用同CoDE类似的方法，即选取多个变异策略进行迭代，改变自适应度的同时对选取的几种变异策略的选用概率进行自适应改变。 
以下为我们选取的四种变异策略 
![](https://img-blog.csdnimg.cn/img_convert/73fe06fb74870917566a986b1cb74ebc.png) 
# 代码实现 
```
close all;
clear all;
clc;
%--------------------初始化各参数值------------------------------------------------------
cecnum=1;
NP=100;
CRm=0.5;
G=500;
LP=100;
D=30;
a=-100;
b=100;
Aa=1;
A=[];
ikun1=0.25;
ikun2=0.25;
ikun3=0.25;
ikun4=0.25;
c=0;
t=0;
r=0;
l=0;
ck=0.000000001;
x=rand(NP,D)*(b-a)+a;        %随机生成初始种群
v=zeros(NP,D);
u=zeros(NP,D);
minx=99999;
best=0;
for i=1:NP                %求各个个体的适应度并取出适应度最好的个体的位置
    fit(i)=cec17(x(i,:),cecnum);
    if fit(i)<minx
        minx=fit(i);
        best=i;
    end
end
trace(1)=minx;
for g=1:G                %开始迭代
    num=randsrc(1,1,[1 2 3 4;ikun1 ikun2 ikun3 ikun4]);        %对选取的策略按照相应概率进行选取
    F=normrnd(0,0.33333)*0.1+0.4;                        %生成符合正态分布的F值
    CR=normrnd(CRm,0.1);        %生成符合正态分布的CR值
%--------------------变异操作----------------------------------------------------------------
    for m=1:NP
       num1=randperm(NP,1);
       while num1==m
           num1=randperm(NP,1);
       end
       num2=randperm(NP,1);
       while (num2==m)||(num2==num1)
           num2=randperm(NP,1);
       end
       num3=randperm(NP,1);
       while (num3==m)||(num3==num1)||(num3==num2)
           num3=randperm(NP,1);
       end
        num4=randperm(NP,1);
       while (num4==m)||(num4==num1)||(num4==num2)||(num4==num3)
           num4=randperm(NP,1);
       end
       num5=randperm(NP,1);
       while (num5==m)||(num5==num1)||(num5==num2)||(num5==num3)||(num5==num4)
           num5=randperm(NP,1);
       end
       if num==1
           v(m,:)=x(num1,:)+F*(x(num2,:)-x(num3,:));
       end
       if num==2
           v(m,:)=x(best,:)+F*(x(num1,:)-x(num2,:))+F*(x(num3,:)-x(num4,:));
       end
       if num==3
           v(m,:)=x(num1,:)+F*(x(num2,:)-x(num3,:))+F*(x(num4,:)-x(num5,:));
       end
       if num==4
           v(m,:)=x(m,:)+F*(x(best,:)-x(m,:))+F*(x(num1,:)-x(num2,:))+F*(x(num3,:)-x(num4,:));
       end
    end
%--------------------交叉操作--------------------------------------------------------
    for n=1:1:D 
        for m=1:NP
           cr=rand;
           if(cr<=CR)     
                u(m,n)=v(m,n);
           else
                u(m,n)=x(m,n);
           end
        end
    end
%--------------------修补染色体--------------------------------------------------------
    for m=1:1:NP        
        for n=1:1:D
            if u(m,n)<=a
                u(m,n)=a;
            end
            if u(m,n)>=b
                u(m,n)=b;
            end
        end
    end
%--------------------重新求适应度，优胜劣汰----------------------
    for i=1:NP
        fit1(i)=cec17(u(i,:),cecnum);
        if fit1(i)<fit(i)
            fit(i)=fit1(i);
            x(i,:)=u(i,:);
        end
        if fit1(i)<minx
            x(i,:)=u(i,:);
            A(1,Aa)=CR;
            Aa=Aa+1;
            minx=fit1(i);
            best=i;
            flag=1;
        end
    end
    if flag==1
        if num==1
                c=c+1;
        end
        if num==2
            t=t+1;
        end
        if num==3
            r=r+1;
        end
        if num==4
            l=l+1;
        end
        flag=0;
    end
    trace(g+1)=minx;
%--------------------对CR进行自适应取值----------------------
    if mod(g,LP)==0
        if ~isempty (A)
            CRm=mean(A(:));                %求所有发生进化的个体所选用的CR的平均值
        end
%--------------------根据所选策略与其成功概率决定以后该策略的选取概率----------------------
        sk1=c/LP+ck;
        sk2=t/LP+ck;
        sk3=r/LP+ck;
        sk4=l/LP+ck;
        ikun1=sk1/(sk1+sk2+sk3+sk4);
        ikun2=sk2/(sk1+sk2+sk3+sk4);
        ikun3=sk3/(sk1+sk2+sk3+sk4);
        ikun4=sk4/(sk1+sk2+sk3+sk4);
        c=0;
        t=0;
        r=0;
        l=0;
    end
    tt=minx;
end
%--------------------结束迭代，绘制迭代次数与适应度的关系图----------------------
x(1,:);
figure(1);
plot(trace);
title(['自适应差分进化算法(SaDE)','最小值',num2str(tt)]);
xlabel('迭代次数');
ylabel('目标函数值');
```
其中调用的函数为cec17测试函数集
# 运行结果 
![](https://img-blog.csdnimg.cn/img_convert/3dca2f10725864900aca1b9e4f81abc9.png)