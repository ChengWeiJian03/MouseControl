# 鼠标调用说明
**此为借助dll文件来调用罗技鼠标驱动，以实现驱动级的键鼠调用**
<br></br>
**注：MouseControl无水印，其他有水印**

### 使用说明
- 驱动：在使用前请安装老版的罗技驱动，即项目文件中的罗技驱动，如安装了新版本的罗技驱动，先卸载干净再安装老版驱动

- 使用前设置：在使用前必须在鼠标设置里，将提高鼠标精度按钮点掉，处于未选中状态，并且将鼠标灵敏度调到正中间，不然鼠标在移动会有偏移
  
- 使用：每个文件完成的键鼠移动都会有误差，每个单位移动的误差大概在0.2-0.3左右，积累起来还是挺大的，如想鼠标到达预定位置建议配合pid控制，示例见文尾
<br></br>

**Python使用DLL文件示例**

```Python
driver = ctypes.CDLL(r'.\MouseControl.dll')
```
### DLL文件使用示例

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
def r_linear_interpolation(r_x,r_y,num_steps,delay):
    r_y = 0-r_y
    dx = r_x/num_steps
    dy = r_y/num_steps
    for i in range(1,num_steps+1):
        next_x,next_y = (dx),(dy)
        driver.move_R(int(next_x),int(next_y))
        time.sleep(delay)
```

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

**Ghub.dll导出函数**

```C++
BOOL INIT() //初始化ghub
void MoveR(int x, int y) //相對移動
void FREE() //釋放
```

**PID控制鼠标移动**
```Python
# Encoding: utf-8
from simple_pid import PID
import ctypes
import pynput

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
        code: 'a'-'z':A键-Z键, '0'-'9':0-9
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

# 鼠标移动
def mouse_move(target_x,target_y):
    while True:
        if abs(target_x - mouse.position[0])<3 and abs(target_y - mouse.position[1])<3:
            break
        pid_x = PID(0.25, 0.01, 0.01, setpoint=target_x)
        pid_y = PID(0.25, 0.01, 0.01, setpoint=target_y)
        next_x = pid_x(mouse.position[0])
        next_y = pid_y(mouse.position[1])
        print(f"next_y={int(round(next_y))}")
        Logitech.mouse.move(int(round(next_x)), int(round(next_y)))
        print(mouse.position)

# 载入驱动
driver = ctypes.CDLL(f'./logitech.driver.dll')
ok = driver.device_open() == 1
mouse = pynput.mouse.Controller() # pynput效率应该是比pyautogui高点

if __name__ == '__main__':
    mouse_move(800,900)
    error = pyautogui.position() # 这里摆个这个pyautogui.position()，是因为pynput好像是有点玄学bug，目标y值超过900鼠标就会挪不过去，但此时只要用一下pyautogui.position()，问题就解决了

```
