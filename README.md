# 正文
>上午开题答辩，被机电系的毕设主任批的昏头晃脑的，不过貌似说的都是大实话，让我的毕设有了阶段性的突破性认知，我现在是个什么状况呢？计算机老师不愿意管我的毕设，昨天去找他签字，说了句：“你快做完，暑假早点过来！”，听这个意思，就是，我不管你的毕设了，你快点做完速度过来。不过也没辙，工业大数据确实我们实验室没这个方向。机电系这边我挂个名义上的老师，结果人都不知道在哪儿，根本不存在指导的问题。so我现在是姥姥不疼，奶奶不爱。今天被胡老师点评一番反而让我坚定了一些想法，美哉！不过，还是先做完机器视觉的作业吧！

![](http://upload-images.jianshu.io/upload_images/3810775-2e18e10070d7ef40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 正文

对产品中心的检测：设置好路径之后，包含关系是在main（相关代码见我以前的一篇文章[【机器视觉与图像处理】基于MATLAB+Hough的圆检测](https://www.jianshu.com/p/8ecd6cfae713)）中调用hough_circle.m 
>示例如下：
```java
>> main

---------------圆统计----------------
  检测出1个圆
  圆心     半径
1 （528，728）  509
Warning: Image is too big to fit on screen; displaying at 67% 
> In images.internal.initSize (line 71)
  In imshow (line 332)
  In main (line 55) 
>>
```

![](http://upload-images.jianshu.io/upload_images/3810775-f8b67ea9606efece.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于闷头的中心的检测、颜色识别，以及数字的识别设置好路径后，包含关系是：MenTou_Color_Number_Detection调用 Color_Depart.m和Tiao.m，
>示例如下：
```java
>>MenTou_Color_Number_Detection
---------------圆统计----------------
  检测出1个圆
  圆心     半径
1 （72，72）  60

检测出的颜色R:255
  
检测出的颜色G:227
  
检测出的颜色B:61

数字是：11
>>
```

![](http://upload-images.jianshu.io/upload_images/3810775-f38c773d16b3b57f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在本次课程设计中，因为数字识别较为困难，所以改为条形码识别。分别设置1到15条长间隔相同宽度相同的条形码（示例如下）：

![](http://upload-images.jianshu.io/upload_images/3810775-2e18e10070d7ef40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在程序中将其转化为二值化图像之后具有良好的识别效果。以上图的11条形码来进行检测迅速得出结果是11。

![](http://upload-images.jianshu.io/upload_images/3810775-35317277a0ea7db2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当然这种用法，我们需要满足一定的假设。首先是闷头上面不应该有杂质或者是别的会影响检测误差的因素。这一点需要工作人员手工实现。另外，我们对照片的要求质量比较高，所以可以采用一个光电门检测产品是否到来。当闷头中心正对镜头的时候拍照最好。

另外就是我们识别出来的颜色是以RGB 三原色来表示，所以在设计过程中需要首先录入15种预设颜色的RGB参数。然后获得当前参数后进行比对即可获得产品颜色数据。

下面是MenTou_Color_Number_Detection.m的代码：

```java
% 文件1 MenTou_Color_Number_Detection.m  
clc;
clear;  
S = imread('/Users/zhangzhaobo/program/MATLAB/Machine_vision/mentou.jpg');  
circleParaXYR=[];  
%取整张图的三维尺寸
[m,n,l] = size(S);  

% 通过判断对象类型来决定是否转化为灰度图
if l>1  
    I = rgb2gray(S); 
else
    I=S
end  

%采用sobel算子来进行边缘检测
BW = edge(I,'sobel');  
[m,n]=size(BW);
% 步长为1，即每次检测的时候增加的半径长度
step_r = 1;  

%检测的时候每次转过的角度
step_angle = 0.1; 

% 对检测的圆的大小范围预估，在实际项目中因为产品大小固定，所以可以给定较小范围，提高运行速度 
minr = 55;  
maxr = 70;  

% 自动取最优的灰度阈值
thresh = graythresh(I);  

% 调用hough_circle函数进行霍夫变换检测圆
[hough_space,hough_circle,para] = hough_circle(BW,step_r,step_angle,minr,maxr,thresh);  
 figure(1),imshow(I),title('原图')  
 figure(2),imshow(BW),title('边缘')  
 figure(3),imshow(hough_circle),title('检测结果')  


circleParaXYR=para;  

%输出  
fprintf(1,'\n---------------圆统计----------------\n');  
[r,c]=size(circleParaXYR); % r=size(circleParaXYR,1);  
fprintf(1,'  检测出%d个圆\n',r); % 圆的个数  
fprintf(1,'  圆心     半径\n'); % 圆的个数  
for n=1:r  
%     x0=floor(circleParaXYR(n,1));
%     y0=floor(circleParaXYR(n,2));
%     if x0>0.25*m && x0<0.75*m && y0>0.25*n && y0<0.75*n
        fprintf(1,'%d （%d，%d）  %d\n',n,floor(circleParaXYR(n,1)),floor(circleParaXYR(n,2)),floor(circleParaXYR(n,3))); 
%    end
end  
  
%标出圆  
 figure(4),imshow(I),title('检测出图中的圆')  
%figure(1),imshow(I),title('检测出图中的圆')  
hold on;  

plot(circleParaXYR(:,2), circleParaXYR(:,1), 'r+');  
for k = 1 : r %size(circleParaXYR, 1)  
    t=0:0.01*pi:2*pi;  
    x=cos(t).*circleParaXYR(k,3)+circleParaXYR(k,2);
    y=sin(t).*circleParaXYR(k,3)+circleParaXYR(k,1);  
    plot(x,y,'r');  
end  

x0=circleParaXYR(1,1);
y0=circleParaXYR(1,2);
r=circleParaXYR(1,3);
[Rr,Gg,Bb]=Color_Depart(S,x0,y0,r);
fprintf('\n检测出的颜色R:%d\n',int32(Rr)); % 
fprintf('  \n检测出的颜色G:%d\n',Gg); %   
fprintf('  \n检测出的颜色B:%d\n',Bb); % 
num=[];
for k=1:11
    num(k)=Tiao(S,x0+5-k,y0,r);
end
fprintf('\n数字是：%d',mode(num));

% 条纹的个数  
fprintf('\n');
```

下面是Color_Depart.m的代码：
```python
function [R,G,B]=Color_Depart(I,x0,y0,r)
% I=imread('Alan_Walker.jpg');
n=r-10;
for k=1:1000
    a = round(x0+(cos(pi/1000*k))*n);
    b = round(y0+(sin(pi/1000*k))*n);
    A(k)=I(a,b,1);
    X(k)=I(a,b,2);
    C(k)=I(a,b,3);
end
R=mode(A);
G=mode(X);
B=mode(C);
```
下面是Tiao.m的代码：
```python
function [number]=Tiao(bw,x0,y0,r)
bw=im2bw(bw);
% [m,n]=size(bw);
number=0;
count=0;
for s=y0-r*0.8:y0+r*0.8
    if bw(x0,s)==0
        count=count+1;
    end
    if count>2 && bw(x0,s)==1 && bw(x0,s+2)==1
        count=0;
        number=number+1;
    end
end
% return number
 ```

说明：Tiao.m与Color_Depart.m都是在MenTou_Color_Number_Detection.m中调用，所以把这些文件全部建立丢到一个文件夹下，设置为工作路径即可按照开头的说明直接用了。

既然都到这儿了，顺手把图贴出来吧！！

![](http://upload-images.jianshu.io/upload_images/3810775-1940073e10b9413d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![记得把这个保存为bmp格式，或者你把前一篇文章中的代码改一下读取文件的名字哈！](http://upload-images.jianshu.io/upload_images/3810775-492544b73990fcf6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 正文之后

>好吧，之所把程序说明写上来，是因为，我在等上传！太多了东西！！！

![](http://upload-images.jianshu.io/upload_images/3810775-de3835bfd252566c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

