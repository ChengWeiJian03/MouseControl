# 鼠标调用说明
<br></br>
**此为借助dll文件来调用罗技鼠标驱动，以实现驱动级的键鼠调用，顺便写了贝塞尔移动，凑合凑合用吧**
<br></br>
### 注：MouseControl无水印，其他有水印，在使用前必须在鼠标设置里，将提高鼠标精度按钮点掉，处于未选中状态，并且将鼠标灵敏度调到正中间，不然鼠标在移动会有偏移
<br></br>
**Python使用DLL文件**

```Python
driver = ctypes.CDLL(r'.\MouseControl.dll')
```
<br/>

### 文件使用示例
<br></br>
**MouseControl.dll调用示例**

```C++
 // 相对移动
 move_R(int x, int y);
 // 绝对移动
 move_Abs(int x, int y);

 // 左键按下
 click_Left_down();
 // 左键松开
 click_Left_up();

 // 右键按下
 click_Right_down();
 // 右键松开
 click_Right_up();
```
<br></br>
**MouseControl.dll平滑移动示例**

```Python
import time
import pyautogui
def linear_interpolation( x_end, y_end, num_steps, delay):# 绝对平滑移动
    start_x, start_y = pyautogui.position()
    dx = (x_end - start_x) / num_steps
    dy = (y_end - start_y) / num_steps

    for i in range(1,num_steps+1):
        next_x = int(start_x + dx * i)
        next_y = int(start_y + dy * i)

        driver.move_Abs(int(next_x), int(next_y))

        time.sleep(delay)

time.sleep(2)
x_end = 30
y_end = 30
root = os.path.abspath(os.path.dirname(__file__))
driver = ctypes.CDLL(r'E:\yolov5-master\MouseControl.dll')
linear_interpolation(x_end, y_end, num_steps=20, delay=0.01)
```

```Python
def r_linear_interpolation(r_x,r_y,num_steps,delay):# 相对平滑移动
    r_y = 0-r_y
    dx = r_x/num_steps
    dy = r_y/num_steps
    for i in range(1,num_steps+1):
        next_x,next_y = (dx),(dy)
        driver.move_R(int(next_x),int(next_y))
        time.sleep(delay)

```
<br></br>
**ghub_device.dll调用示例**

```Python
from ctypes import CDLL

try:
    gm = CDLL(r'./ghub_device.dll')
    gmok = gm.device_open() == 1
    if not gmok:
        print('未安装ghub或者lgs驱动!!!')
    else:
        print('初始化成功!')
except FileNotFoundError:
    print('缺少文件')

#按下鼠标按键
def press_mouse_button(button):
    if gmok:
        gm.mouse_down(button)

#松开鼠标按键
def release_mouse_button(button):
    if gmok:
        gm.mouse_up(button)

#点击鼠标
def click_mouse_button(button):
    press_mouse_button(button)
    release_mouse_button(button)

#按下键盘按键
def press_key(code):
    if gmok:
        gm.key_down(code)

#松开键盘按键
def release_key(code):
    if gmok:
        gm.key_up(code)

#点击键盘按键
def click_key(code):
    press_key(code)
    release_key(code)

# 鼠标移动
def mouse_xy(x, y, abs_move = False):
    if gmok:
        gm.moveR(int(x), int(y), abs_move)
```
<br></br>
**logitech.driver.dll调用示例**

```Python
import ctypes
import os
import pynput
import winsound

try:
    root = os.path.abspath(os.path.dirname(__file__))
    driver = ctypes.CDLL(f'{root}/logitech.driver.dll')
    ok = driver.device_open() == 1  # 该驱动每个进程可打开一个实例
    if not ok:
        print('Error, GHUB or LGS driver not found')
except FileNotFoundError:
    print(f'Error, DLL file not found')


class Logitech:

    class mouse:

        """
        code: 1:左键, 2:中键, 3:右键
        """

        @staticmethod
        def press(code):
            if not ok:
                return
            driver.mouse_down(code)

        @staticmethod
        def release(code):
            if not ok:
                return
            driver.mouse_up(code)

        @staticmethod
        def click(code):
            if not ok:
                return
            driver.mouse_down(code)
            driver.mouse_up(code)

        @staticmethod
        def scroll(a):
            """
            a:没搞明白
            """
            if not ok:
                return
            driver.scroll(a)

        @staticmethod
        def move(x, y):
            """
            相对移动, 绝对移动需配合 pywin32 的 win32gui 中的 GetCursorPos 计算位置
            pip install pywin32 -i https://pypi.tuna.tsinghua.edu.cn/simple
            x: 水平移动的方向和距离, 正数向右, 负数向左
            y: 垂直移动的方向和距离
            """
            if not ok:
                return
            if x == 0 and y == 0:
                return
            driver.moveR(x, y, False)

    class keyboard:

        """
        键盘按键函数中，传入的参数采用的是键盘按键对应的键码
        code: 'a'-'z':A键-Z键, '0'-'9':0-9, 其他的没猜出来
        """

        @staticmethod
        def press(code):

            if not ok:
                return
            driver.key_down(code)

        @staticmethod
        def release(code):
            if not ok:
                return
            driver.key_up(code)

        @staticmethod
        def click(code):
            if not ok:
                return
            driver.key_down(code)
            driver.key_up(code)


if __name__ == '__main__':  # 测试

    winsound.Beep(800, 200)

    def release(key):
        if key == pynput.keyboard.Key.end:  # 结束程序 End 键
            winsound.Beep(400, 200)
            return False
        elif key == pynput.keyboard.Key.home:  # 移动鼠标 Home 键
            winsound.Beep(600, 200)
            Logitech.mouse.move(100, 100)

    with pynput.keyboard.Listener(on_release=release) as k:
        k.join()

```
<br></br>
**Ghub.dll导出函数**

```C++
BOOL INIT() //初始化ghub
void MoveR(int x, int y) //相對移動
void FREE() //釋放
```
<br></br>

### 贝塞尔移动示例
</br>
**一阶贝塞尔移动**
```Python
import random
import time
import ctypes
import matplotlib.pyplot as plt
import numpy as np
import pyautogui

class bezer:
    driver = None
    points = None
    B1 = None
    t = None

    def __init__(self):
        self.driver = ctypes.CDLL(r'.\MouseControl.dll')

    def sum_decimal_places(self,deff_array, decimal_places=5):
        # 将数组舍入到指定的小数位数
        rounded_arr = np.round(deff_array, decimal_places)

        # 提取小数部分
        decimals = np.modf(rounded_arr)[0]

        # 将小数部分乘以相应的权重，再进行求和
        weighted_sum = np.round(np.sum(decimals,axis=0))

        return weighted_sum

    def getPoint(self, start, end):
        self.points = np.array([[start[0], start[1]],[end[0], end[1]]])  # 在此处修改坐标
        t = np.linspace(0, 1, 100)
        t = np.array([t, t]).T
        self.B1 = (1 - t) * self.points[0] + t * self.points[1]
    def absMove(self,point=(1920,1080),delay=0): 
        start = pyautogui.position()
        self.getPoint(start, point)
        for x, y in zip(self.B1[:, 0], self.B1[:, 1]):
            self.driver.move_Abs(int(x), int(y))
            time.sleep(delay)
    def eliminatingErrors(self,diff_arr):
        sum_array = np.round(self.sum_decimal_places(np.abs(diff_arr), 5))
        max = np.max(sum_array).astype(int)
        wuchashuzu = np.full((max, 2), 0)
        if diff_arr[0][0]<0:
            wuchashuzu[:int(sum_array[0]), 0] = -1 # TODO
        else:
            wuchashuzu[:int(sum_array[0]), 0] = 1
        if diff_arr[0][1]<0:
            wuchashuzu[:int(sum_array[1]),1] = -1
        else:
            wuchashuzu[:int(sum_array[1]),1] = 1
        return wuchashuzu

    def rMove(self,point,delay=0):
        i = 0
        start = pyautogui.position()
        self.getPoint(start, point)
        self.B1 = np.abs(self.B1)
        diff_arr = np.diff(self.B1,axis=0)
        error = self.eliminatingErrors(diff_arr)
        int_arr = diff_arr.astype(int)
        int_arr = np.concatenate((int_arr,error))
        for x,y in zip(int_arr[:,0],int_arr[:,1]):
            i=i+1
            self.driver.move_R(int(x),int(y))

    def show(self):
        plt.plot(self.B1[:, 0].astype(int), self.B1[:, 1].astype(int))
        plt.show()

bezer1 = bezer()

while True:
    pass
    # random.randint(1920),random.randint(1080)
    # bezer1.absMove((320,221),0)
    # print(pyautogui.position())
    # bezer1.rMove((0,0))
```
<br></br>
**二阶贝塞尔移动示例**
```Python
import time
import ctypes
import matplotlib.pyplot as plt
import numpy as np
import pyautogui

class bezer:
    driver = None
    n = None
    init_t = None
    P = None
    points = None
    def __init__(self):
        self.driver = ctypes.CDLL(r'.\MouseControl.dll')

    def getB(self,i):
        t = np.math.factorial(self.n) * self.init_t ** i * (1 - self.init_t) ** (self.n - i) / (
                    np.math.factorial(i) * np.math.factorial(self.n - i))
        return np.array([t, t]).T
    def GetPoint(self,start,end):
        self.points = np.array([[start[0], start[1]], [960,540], [end[0], end[1]]])  # 在此处修改坐标
        self.n = self.points.shape[0] - 1
        self.init_t = np.linspace(0, 1, 1000)
        self.P = np.zeros((1000, 2))
        for i in range(self.n + 1):
            self.P += self.getB(i) * self.points[i]
    def move(self,start=(0,0),end=(1920,1080),delay=0):
        self.GetPoint(start,end)
        i=0
        for x, y in zip(self.P[:, 0], self.P[:, 1]):
            i+=1
            self.driver.move_Abs(int(x), int(y))
            time.sleep(delay)
    def show(self):
        plt.plot(self.P[:, 0].astype(int), self.P[:, 1].astype(int))
        plt.show()

bezer1 = bezer()

while True:
    start_x, start_y = pyautogui.position()
    bezer1.move((start_x,start_y),(430,650),0)
    time.sleep(1)

```
