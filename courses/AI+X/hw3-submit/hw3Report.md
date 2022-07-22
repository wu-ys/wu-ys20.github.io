# AI+X Homework 3 Report

## Task 1 : LED

源代码都有了，按照教程做就行了，基本没有坑。

## Task 2 : Key

约束文件和上面类似，找到按键开关和LED灯泡对应的管脚号即可，注意驱动电压都是是$3.3V$

## Task 3 : ILA Example

1. 如何创建example design：新建一个project，在IP Catalog里面新建一个ILA（默认设置即可），之后在Sources页面切换到IP Source栏，右击这个ILA IP，找到Open Example Design

![image-20220722113727488](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722113727488.png)

2. 需要做的修改：在xdc文件中修改时钟引脚

Schematic图：

![image-20220722120646190](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722120646190.png)

ILA输出波形如下：

![image-20220722120619023](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722120619023.png)

## Task 4 : PLL

使用PLL IP，可以实现时钟周期转换。

#### 遇到的和困难注意点

1. ILA的采样时钟需要是free running clock。经过仿真波形可以看到，PLL起初需要大约300纳秒时间才能输出稳定的信号，因此用PLL生成的时钟无法直接作为ILA的取样时钟。
2. ILA采样周期和时钟周期如果没有整数倍关系的话，ILA会输出不均匀的的波形。另外，使用200Hz的采样频率无法对200Hz的信号采样，建议可以使用更高频的时钟。

#### 仿真波形

Schematic图见pdf文件。仿真波形如下：

![image-20220722145235476](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722145235476.png)

以下分别为200Hz采样频率和500Hz采样频率下的ILA波形。

![200Hz采样频率](D:\wys\classfile\2021-2022-3\AI+X\codes\hw3-submit\hw3Report.assets\image-20220722142028739.png)

![500Hz采样频率](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722142139422.png)



## Task 5 : ROM & RAM

#### C++测试文件

用C++生成两位十六进制随机数：

```c++
// rom_init.cpp
#include <iostream>
#include <cmath>

int main() {
    srand((unsigned)time(0));
    std::cout << "MEMORY_INITIALIZATION_RADIX=16;\n";
    std::cout << "MEMORY_INITIALIZATION_VECTOR=\n";
    char chars[16] = {'0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f'};

    for (int i = 0; i < 32; i++) 
    {
        char char1 = chars[rand() % 16];
        char char2 = chars[rand() % 16];
        std::cout << char1 << char2 << ((i==31)?';':',') << std::endl;
    }
    return 0;

}
```

通过重定向分别写入`rom1.coe1`和`rom2.coe`。读取文件生成golden reference：

```c++
// rom_prod.cpp
#include <iostream>
#include <cmath>
#include <fstream>
#include <string>
using namespace std;
int convert(char s);
int main() {
    ifstream rom1;
    rom1.open("rom1.coe");
    ifstream rom2;
    rom2.open("rom2.coe");
    char char1;
    char char2;
    string s1;
    getline(rom1, s1);
    getline(rom1, s1);
    getline(rom2, s1);
    getline(rom2, s1);
    int sum = 0;
    
    for (int i = 0; i < 32; i++) {
        rom1.get(char1);
        rom1.get(char2);

        int op1 = convert(char1) * 16 + convert(char2);
        cout << op1 << ' ';

        rom2.get(char1);
        rom2.get(char2);

        int op2 = convert(char1) * 16 + convert(char2);
        cout << op2 << ' ';
        sum = op1*op2;

        cout << sum << endl;

        getline(rom1, s1);
        getline(rom2, s1);
    }
    rom1.close();
    rom2.close();
    return 0;

}

int convert(char s) {
    if ('0' <= s && s <= '9')
        return s-48;
    switch (s) {
    case 'a':
        return 10; break;
    case 'b':
        return 11; break;
    case 'c':
        return 12; break;
    case 'd':
        return 13; break;
    case 'e':
        return 14; break;
    case 'f':
        return 15; break;
    default:
        return 0; break;
    }
}
```

ROM初始化完成后，每个周期从两个ROM中读取一个数据，做乘法后写进RAM的对应位置。最后不断依次读取RAM中的数据并用ILA输出波形。

#### 遇到的困难和注意点

1. 乘法逻辑较为复杂，如果直接使用乘号计算，在200MHz的高频时钟下可能会出现时序问题。偷懒的解决办法是调高时钟周期（？），不过更推荐的方法是使用一个Multiplier IP来计算，可以设置在若干个周期的延迟后输出，也可以进行流水线作业，以匹配200MHz的时钟。
2. ROM会在输入地址后的第二个clk上升沿输出数据，需要注意数据和地址的匹配。

#### 仿真波形

1. 初始状态：

![sim5](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/sim5-01.jpg)

2. RAM存储完成后，开始读取：

![sim52](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/sim52-01.jpg)

Schematic图见pdf文件。

#### Utilization Summary

这里因为把RAM的32位输出全部接到了IO，所以对IO的占用比较大。

![image-20220722161200683](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722161200683.png)

#### ILA波形

与此前仿真中预测的一致，延迟两个周期输出数据。

![image-20220722161327997](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722161327997.png)

## Task 6 : FIFO

FIFO模块在`wr_en`信号为高时读入数据，在`rd_en`信号为高时输出数据，同时提供`full`, `empty`, `data_count`等相应接口；另外，注意复位信号`srst`是**高电平**有效。

#### 遇到的困难和注意点

1. 注意复位信号高电平有效……在写仿真和上板的时候都犯了这个错误。
2. 样例中给的组合逻辑基本都可以借鉴，不过最好把触发条件改为`posedge sys_clk`，同时还需要加上更新状态的赋值语句等必要的补充。

#### 仿真波形

![image-20220722164151149](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722164151149.png)

第一部分中，我们保持`wr_en`信号为高，依次输入0~255的数据，可见`data_count`信号同步增长，在最后一个数$255$写入后的第一个周期，`data_count`达到$256$（由于只有八位，所以显示为零），`full`信号变高，从此开始切换为输出模式。下一个周期正常输出了FIFO中存储的第一个数$0$.

Schematic图见pdf文件。

#### Utilization Summary

![image-20220722164743626](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722164743626.png)

#### ILA波形

与仿真模拟中预测的一致。不过由于组合逻辑的原因，`rd_en`和`wr_en`信号的切换延迟了一个周期。

从写到读：

![image-20220722164934446](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722164934446.png)

从读到写：

![image-20220722165117389](https://wu-ys.github.io/courses/AI+X/hw3-submit/hw3Report.assets/image-20220722165117389.png)