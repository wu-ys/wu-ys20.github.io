# 用64行代码在电脑上画一个火箭

代码仓库：https://github.com/wu-ys/rocket-voxel

由于本人是又菜又爱玩的MC玩家（比不上长满了肝的建造大佬），喜欢用方块搭东西但又懒得自己一个个搭。最近学了Taichi Lang，正好看到了Taichi的胡渊鸣写的体素艺术的简单框架：https://mp.weixin.qq.com/s?__biz=MzkzNDI3NDY4Mw==&mid=2247493732&idx=1&sn=8ae0fd731181aa82024f69eba1f75fa7&chksm=c2bd1b70f5ca9266efc7b71bdf601fbd975dee040ef8b173a46ec4418580972c558628529b5f&cur_album_id=2331136439461494785&scene=189#wechat_redirect（模板仓库见https://github.com/taichi-dev/voxel-challenge）

正值神舟十五号发射，于是心血来潮花了大约3个小时画了一个正在发射的火箭，同时为正在上的另一门课（个性化3D设计与实现）交了一幅图案。效果如下：

![image-20221129105535044](D:\wys\github-website\wu-ys20\thoughts\rocket.assets\image-20221129105535044.png)

分步骤讲解：

1. [火箭主体：圆柱](###圆柱体)

2. [火箭主体：整流罩](###整流罩)

3. [3D字样](##装饰)

4. [尾焰](###尾焰)

5. [背景、灯光、分辨率、曝光等等其他参数](##其他参数)

## 火箭主体绘制

### 圆柱体

写一个绘制圆柱体的函数

```python
@ti.func
def draw_cylinder(pos, radius, height, color, color_noise):
```

五个参数分别是（底面）圆心坐标，半径，圆柱体高度，颜色，颜色噪声限。画图的代码非常简单：

```python
@ti.func
def draw_cylinder(pos, radius, height, color, color_noise):
  for I in ti.grouped(ti.ndrange((-radius, 1+radius), (0, height), (-radius, 1+radius))):
    if (I[0]**2 + I[2]**2 <= radius**2):
      scene.set_voxel(pos+I, 1, color + color_noise* ti.random())
```

这里为了充分利用Taichi的并行化特性，直接让最外层循环跑遍$[-r,r]\times[0,h)\times[-r,r]$的范围，判断$x^2+z^2\le r^2$。

### 整流罩

这里将整流罩的曲面抽象为抛物面$z = c -k(x^2+y^2)$，在抛物面以下的部分我们去用体素填满。

```python
@ti.func
def draw_cone(pos, radius, height, color, color_noise):
  for I in ti.grouped(ti.ndrange((-radius, 1+radius), (0, height), (-radius, 1+radius))):
    if (I[0]**2 + I[2]**2 <= radius**2 * (1- I[1]/height)):
      if (I[1] == 0):
        scene.set_voxel(pos+I, 1, vec3(0.1, 0.1, 1.0))
      else:
        scene.set_voxel(pos+I, 1, color + color_noise*ti.random())  
```

五个参数分别是（底面）圆心坐标，半径，圆柱体高度，颜色，颜色噪声限。其中我们将底面的圆周（也就是`I[1]==0`）涂成蓝色，更加贴近现实的火箭（至少中国的涂装是这样的）

### 尾焰

还有很重要的一点，那就是火箭的尾焰。看现实的照片，火箭尾焰基本是以一个角度向外散射，越往后越模糊。

![img](D:\wys\github-website\wu-ys20\thoughts\rocket.assets\u=2648878364,2505082198&fm=253&fmt=auto&app=120&f=JPEG.jpeg)

我们可以用这一方法，模拟出一定高度内的尾焰，并指定其喷射范围；另外，我们用一个$U[0,1]$的随机数与$(1-h/h_0)^2$比较，来为不同高度的每个点上是否有体素赋上一定的概率，让尾焰看起来更有喷射的样子。

```python
@ti.func
def draw_fire(pos, radius, height, color, color_noise):
    for I in ti.grouped(ti.ndrange((-2*radius, 1+2*radius), (0, height), (-2*radius, 1+2*radius))):
        if (I[0]**2 + I[2]**2 <= radius**2 * (1+3*I[1]/height)) and (ti.random() < (1- I[1]/height)**2):
            scene.set_voxel(vec3(pos[0]+I[0], pos[1]-I[1], pos[2]+I[2]), 20, color + color_noise * ti.random()) 
```

怎么样，看起来还有点样子吧：

![image-20221129004812897](D:\wys\github-website\wu-ys20\thoughts\rocket.assets\image-20221129004812897.png)



## 装饰

因为是3D打印课的图样，所以在火箭上写了3D的字样，另外画了一面祖传国旗（五颗星真的画不上了……）

![image-20221129105512938](D:\wys\github-website\wu-ys20\thoughts\rocket.assets\image-20221129105512938.png)

3和D两个字符是通过打表的方式，逐方块印在表面上：

```python
# python scope
char3=[[2,1], [1,2], [1,3], [1,4], [1,5], [2,6], [3,7], [4,7], [5,6], [6,5], [6,4], [7,6], [8,7], [9,7], [10,6], [11,5], [11,4], [11,3], [11,2], [10,1]]
char_3 = ti.Matrix(char3)
chard = [[1,1], [2,1], [3,1], [4,1], [5,1], [6,1], [7,1], [8,1], [9,1], [10,1], [11,1], [1,2], [1,3], [2,4], [2,5], [3,6], [4,7], [5,7], [6,7], [7,7], [8,7], [9,6], [10,5], [10,4], [11,2], [11,3]]
char_d = ti.Matrix(chard)

# taichi scope
# draw 3
for key in ti.static(range(20)):
  scene.set_voxel(vec3(-4+char_3[key,1], 130+char_3[key,0], 10), 1, vec3(0.1, 0.1, 1.0))
# draw d
for key in ti.static(range(26)):
  scene.set_voxel(vec3(-4+char_d[key,1], 114+char_d[key,0], 10), 1, vec3(0.1, 0.1, 1.0))
```

国旗则直接用循环画出一个矩形就可以：

```python
# draw flag
for I in ti.static(ti.ndrange((-4,5), (-3,3))):
  scene.set_voxel(vec3(I[0], I[1]+106, 10), 1, vec3(1.0, 0.1, 0.1))
```



## 其他参数

```python
scene = Scene(voxel_edges=0, exposure=3)	# 体素边，设置为0；曝光设置为3，也可以尝试其它数值
scene.set_floor(-10, (0.6, 0.6, 0.6))			# 地面高度-10，颜色 (0.6, 0.6, 0.6)
scene.set_background_color((0.1, 0.4, 0.8))		# 天空颜色 (0.1, 0.4, 0.8)
scene.set_directional_light((0, 1, 1), 0.2, (1, 0.8, 0.6))	# 光照方向、强度和颜色
```

