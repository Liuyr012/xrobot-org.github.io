# 设计麦轮底盘

利用前面的知识，从头开始设计麦克纳姆轮底盘。

## 创建麦轮模块

使用下面命令创建一个mecanum模块

```shell
./project.py new module mecanum
```

然后完善mod_mecanum.cpp和mod_mecanum.hpp文件。开启一个线程用来控制，再定义一个Param结构体用来存放pid等参数

`mod_mecanum.cpp`

```cpp
#include "mod_mecanum.hpp"

using namespace Module;

Mecanum::Mecanum(Param& param) : param_(param) {
  auto thread_fn = [](Mecanum* mecanum) {
    while (true) {
      mecanum->thread_.SleepUntil(2);
    }
  };

  this->thread_.Create(thread_fn, this, "mecanum_ctrl", 512,
                       System::Thread::MEDIUM);
}
```

`mod_mecanum.hpp`

```cpp
#include "module.hpp"

namespace Module {
class Mecanum {
 public:
  struct Param {};

  Mecanum(Param& param);

 private:
  Param& param_;

  System::Thread thread_;
};
}  // namespace Module

```

## 麦轮控制原理

![麦克纳姆轮](../../img/mecanum.jpg)

完整的麦轮底盘需要四个麦克纳姆轮，以O型的方式安装。在小轮的作用下，每个轮子对底盘的力的作用方向和轮子的转动方向呈45度角。

这里轮子顺序按照图中编号，箭头所指y轴正方向为车的正前方，车身顺时针旋转为z轴。

```txt
   ②   y轴正方向    ①
  ///      ↑       \\\
  ///-|    ↑     |-\\\
  /// |    ↑     | \\\
      |          |
      |----------|   ---------> x轴正方向
      |          |
  \\\ |          | ///
  \\\-|          |-///
  \\\              ///
   ③               ④
```

如果运动向量为xyz，那么每个轮子的目标速度应当为

```cpp
speed_1 = x - y + z;
speed_2 = x + y + z;
speed_3 = -x + y + z;
speed_4 = -x - y + z;
```

## 底盘模式

```cpp
typedef enum {
    RELAX,         /* 放松模式，电机不输出。一般情况底盘初始化之后的模式 */
    FOLLOW_GIMBAL, /* 通过闭环控制使车头方向跟随云台 */
    ROTOR,         /* 小陀螺模式，通过闭环控制使底盘不停旋转 */
    INDENPENDENT,  /* 独立模式。底盘运行不受云台影响 */
} Mode;
```

## 电机

为了能使麦轮模块适配多种电机，此处选择在构造时传入BaseMotor指针，不考虑具体电机型号。其他电机都由BaseMotor类派生出来。

## TODO 累了明天再写
