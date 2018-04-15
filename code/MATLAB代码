%程序用于计算跳一跳中需要跳跃的距离与时间


clear;
fclose(instrfind);
sr6 = serial('COM7');       % 使用默认设置创建串口sr6
fopen(sr6);                 %打开串口  
while(1)

%首先完成图像的获取
t = java.awt.Toolkit.getDefaultToolkit();
robo = java.awt.Robot;
rec = java.awt.Rectangle(t.getScreenSize());
image = robo.createScreenCapture(rec);
filehandle = java.io.File('source.png');
javax.imageio.ImageIO.write(image,'png',filehandle);
a=imread('source.png');
a=imcrop(a,[1400,300,600,700]); %与屏幕演示窗口所放置的位置相关

%找小人坐标
%用网上已有的模板匹配程序
template_rgb = imread('model2.png');
src_rgb = a;
%转换为灰度图
template=rgb2gray(template_rgb);    template = im2double(template);
src=rgb2gray(src_rgb);  src = im2double(src);

%球的模板与原始图像的大小
tempSize=size(template);
tempHeight=tempSize(1); tempWidth=tempSize(2);
srcSize=size(src);
srcHeight=srcSize(1); srcWidth=srcSize(2);

%在图片的右侧与下侧补0
srcExpand=padarray(src,[tempHeight-1 tempWidth-1],'post');

%初始化一个距离数组 tmp:mj  template:x
distance=zeros(srcSize);
for height=fix(0.4*srcHeight):fix(0.6*srcHeight)   %将模板匹配的范围缩小从而增大运算速度
   for width= fix(0.20*srcWidth):fix(0.76*srcWidth)  %取的范围为多次实验后的
      tmp=srcExpand(height:(height+tempHeight-1),width:(width+tempWidth-1));
      %计算决策函数
      distance(height,width)=sum(sum(template'*tmp-0.5.*(tmp'*tmp)));
   end
end

%寻找决策函数最大时的索引
maxDis=max(max(distance));
[x_model, y_model]=find(distance==maxDis);


%找目标点坐标
aim=a;
aim=im2double(aim);
[m,n,d]=size(aim);

%取背景点值为左上角的点和最后一行背景点的平均
if(   (aim(1,1,1)-aim(m,1,1))^2+(aim(1,1,2)-aim(m,1,2))^2+(aim(1,1,3)-aim(m,1,3))^2 ...  %避免取的最后一行的点是物体而非背景
      <=...
      (aim(1,1,1)-aim(m,n,1))^2+(aim(1,n,2)-aim(m,n,2))^2+(aim(1,1,3)-aim(m,n,3))^2  )
    bg_r=0.7*aim(1,1,1 )+0.3*aim(m,1,1);   %上半部分权重大一点
    bg_g=0.7*aim(1,1,2 )+0.3*aim(m,1,2);
    bg_b=0.7*aim(1,1,3 )+0.3*aim(m,1,3);
else
    bg_r=0.7*aim(1,1,1 )+0.3*aim(m,n,1);
    bg_g=0.7*aim(1,1,2 )+0.3*aim(m,n,2);
    bg_b=0.7*aim(1,1,3 )+0.3*aim(m,n,3); 

end
        
for i=1:m
    for j=1:n      
        if(    (sqrt( (aim(i,j,1)-bg_r).^2+(aim(i,j,2)-bg_g).^2+(aim(i,j,3)-bg_b).^2 )<0.15)...
              ||(abs(i-x_model+30)<20&&abs(j-y_model-16)<20)  )   
            %如果是背景，则让它变成黑色  头部也变为背景
                aim(i,j,1)=0;
                aim(i,j,2)=0;
                aim(i,j,3)=0;    
        end           
    end
end
aim=im2bw(aim,0.01);   %图像二值化
imshow(aim)
hold on, plot(y_model,x_model,'ro')

%找目标方块坐标  分别用x_aim, y_aim表示
    % 找目标方块纵坐标
    count=0;
    y_sum=0;
    for i=1:m
        for j=1:n
            if(aim(i,j)==1)
                count=count+1;
                y_sum=y_sum+j;
            end         
        end
            if(count>=3)
                y_aim=fix(y_sum/count);
                break;
            end
    end
    
    %找方块横坐标
    x_sum=0;
    x_aim_t=i;
    x_aim=i;
    y_edge=y_aim;
    flag=1;
    x_search_number=0;
    
    while(  aim(x_aim_t,y_edge)==0  ) %确保起始点在目标块上   
          y_aim=y_aim+1;
          y_edge=y_aim;
    end
   while(aim(x_aim_t,y_edge))
       if(y_edge+1<=n&&aim(x_aim_t,y_edge+1))
           if(flag>6)
               break;
           else
           y_edge=y_edge+1;
           x_sum=x_aim_t;
           x_aim=x_aim_t;
           flag=1;
           end
       else
           x_aim_t=x_aim_t+1;
           x_sum=x_sum+x_aim_t;
           flag=flag+1;
           if(flag>=16)
               break;
           end
       end       
   end
   if(flag<16)
       x_aim=fix(x_sum/flag);
   end
      


%绘制匹配结果
imshow(a);title('起点终点寻找结果'),hold on
%rectangle('Position',[y_model x_model tempWidth tempHeight],'LineWidth',2,'LineStyle','--','EdgeColor','r')
hold on,plot(y_aim,x_aim,'ro');

%之前的model为左上角的，以下是取model中心点
x_model=x_model+50;  
y_model=y_model+16;
hold on,plot(y_model,x_model,'ro');
%计算距离以及时间
displacement=sqrt(  (x_aim-x_model).^2 + (y_aim-y_model).^2  );

%press_time = (displacement^0.85)*3.7; %（幂函数拟合），可选，对于要求较高可以选用
%向单片机发送数据
 
  
  delay= uint8(displacement/3+4);   %4为舵机启动到触碰到屏幕的时间，基于不同的硬件实际情况进行修改
  fwrite(sr6,delay);           % 给串口写入数据  
 
  t_delay=double(delay)*0.02+2.5;  %每个delay单片机上实际是20ms
  pause(t_delay);  %等待单片机完成按压以及游戏动画
end
