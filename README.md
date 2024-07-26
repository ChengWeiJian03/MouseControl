# 鼠标调用说明
**此为借助dll文件来调用罗技鼠标驱动，以实现驱动级的键鼠调用**
<br></br>
**注：MouseControl无水印，其他有水印**

### 使用说明
- 驱动：在使用前请安装老版的罗技驱动，即项目文件中的罗技驱动，如安装了新版本的罗技驱动，先卸载干净再安装老版驱动

- 使用前设置：在使用前必须在鼠标设置里，将提高鼠标精度按钮点掉，处于未选中状态，并且将鼠标灵敏度调到正中间，不然鼠标在移动会有偏移
  
- 使用：每个文件完成的键鼠移动都会有误差，每个单位移动的误差大概在0.2-0.3左右，积累起来还是挺大的，如想鼠标到达预定位置建议配合pid控制，示例见文尾

- 芜湖~写了个LADRC控制鼠标移动,虽然不好用,但勉强用用.也在文尾,我甚至贴心的给了可视化调参
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
import pyautogui
def mouse_move(driver,target_x,target_y):
    mouse = pynput.mouse.Controller()
    while True:
        if abs(target_x - mouse.position[0])<3 and abs(target_y - mouse.position[1])<3:
            break
        pid_x = PID(0.25, 0.01, 0.01, setpoint=target_x)
        pid_y = PID(0.25, 0.01, 0.01, setpoint=target_y)
        next_x,next_y = pid_x(mouse.position[0]),pid_y(mouse.position[1])
        driver.moveR(int(round(next_x)), int(round(next_y)), False) # 鼠标移动
        # print(mouse.position) # 打印鼠标位置

if __name__ == '__main__':
    driver = ctypes.CDLL(f'./logitech.driver.dll')
    while 1:
        mouse_move(driver,800,900)
        error = pyautogui.position() # 这里摆个这个pyautogui.position()，是因为pynput好像是有点玄学bug，目标y值超过900鼠标就会挪不过去，但此时只要用一下pyautogui.position()，问题就解决了

```

**LADRC控制鼠标移动**
```Python
import streamlit as st
import matplotlib.pyplot as plt
import numpy as np
from functools import partial
from math import factorial
import pyautogui
import time

from matplotlib import rcParams


class LADRC:
    def __init__(self,
                 ordenProceso: int,
                 gananciaNominal: float,
                 anchoBandaControlador: float,
                 anchoBandaLESO: float,
                 condicionInicial: int
                 ) -> None:
        self.u = 0
        self.h = 0.001

        self.nx = int(ordenProceso)
        self.bo = gananciaNominal
        self.wc = anchoBandaControlador
        self.wo = anchoBandaLESO
        self.zo = condicionInicial

        self.Cg, self.Ac, self.Bc, self.Cc, self.zo, self.L, self.K, self.z = self.ConstruirMatrices()

    def ConstruirMatrices(self) -> tuple:
        n = self.nx + 1

        K = np.zeros([1, self.nx])
        for i in range(self.nx):
            K[0, i] = pow(self.wc, n - (i + 1)) * (
                    (factorial(self.nx)) / (factorial((i + 1) - 1) * factorial(n - (i + 1))))

        L = np.zeros([n, 1])
        for i in range(n):
            L[i] = pow(self.wo, i + 1) * (
                    (factorial(n)) / (factorial(i + 1) * factorial(n - (i + 1))))

        Cg = self.bo

        Ac = np.vstack((np.hstack((np.zeros([n - 1, 1]), np.identity(n - 1))), np.zeros([1, n])))
        Bc = np.vstack((np.zeros([self.nx - 1, 1]), self.bo, 0))
        Cc = np.hstack(([[1]], np.zeros([1, n - 1])))
        zo = np.vstack(([[self.zo]], np.zeros([n - 1, 1])))
        z = np.zeros([n, 1])

        return Cg, Ac, Bc, Cc, zo, L, K, z

    def LESO(self, u, y, z) -> np.ndarray:
        return np.matmul(self.Ac, z) + self.Bc * u + self.L * (y - np.matmul(self.Cc, z))

    def Runkut4(self, F, z, h):
        k0 = h * F(z)
        k1 = h * F(z + 0.5 * k0)
        k2 = h * F(z + 0.5 * k1)
        k3 = h * F(z + k2)
        return z + (1 / 6) * (k0 + 2 * k1 + 2 * k2 + k3)

    def SalidaControl(self, r: int, y: int):
        leso = partial(self.LESO, self.u, y)
        self.z = self.Runkut4(leso, self.z, self.h)
        u0 = self.K[0, 0] * (r - self.z[0, 0])
        for i in range(self.nx - 1):
            u0 -= self.K[0, i + 1] * self.z[i + 1, 0]

        return (u0 - self.z[self.nx, 0]) * self.Cg


def control_movimiento_raton(x_objetivo, y_objetivo, wc, wo, bo):
    controlador = LADRC(ordenProceso=2, gananciaNominal=bo, anchoBandaControlador=wc, anchoBandaLESO=wo,
                        condicionInicial=0)
    trajectory = []
    move_attempts = 0
    max_attempts = 50

    while move_attempts < max_attempts:
        x_actual, y_actual = pyautogui.position()
        trajectory.append((x_actual, y_actual))
        error_x = x_objetivo - x_actual
        error_y = y_objetivo - y_actual

        if abs(error_x) < 5 and abs(error_y) < 5:
            st.write("移动成功")
            break

        u_x = controlador.SalidaControl(error_x, x_actual)
        u_y = controlador.SalidaControl(error_y, y_actual)

        # Debug outputs
        # st.write(f"Attempt: {move_attempts}")
        # st.write(f"Current Position: ({x_actual}, {y_actual})")
        # st.write(f"Control Output: ({u_x}, {u_y})")
        # st.write(f"Errors: ({error_x}, {error_y})")

        pyautogui.moveRel(int(u_x), int(u_y))
        time.sleep(0.01)
        move_attempts += 1

    # Final check for success
    x_final, y_final = pyautogui.position()
    if abs(x_final - x_objetivo) < 5 and abs(y_final - y_objetivo) < 5:
        st.write("移动成功")
    else:
        st.write("移动失败")

    return trajectory


# Streamlit 代码
st.title("LADRC 控制鼠标移动调参")

x_target = st.slider("目标位置 X", 0, 1920, 500)
y_target = st.slider("目标位置 Y", 0, 1080, 500)
wc = st.slider("控制器带宽 (wc)", 0.1, 5.0, 1.0)
wo = st.slider("LESO 带宽 (wo)", 0.1, 5.0, 1.0)
bo = st.slider("增益 (bo)", 0.1, 2.0, 0.7)

if st.button("开始控制"):
    trajectory = control_movimiento_raton(x_target, y_target, wc, wo, bo)

    # 绘制鼠标移动轨迹
    x_values = [point[0] for point in trajectory]
    y_values = [point[1] for point in trajectory]
    rcParams['font.family'] = ['SimHei'] 
    rcParams['axes.unicode_minus'] = False 

    plt.figure(figsize=(10, 6))

    plt.scatter(x_values[0], y_values[0], color='g', label='起点')
    plt.scatter(x_values[-1], y_values[-1], color='r', label='终点')
    plt.scatter(x_target, y_target, color='purple', label='目标点')
    plt.plot(x_values, y_values, marker='o', linestyle='-', color='b')
    plt.xlim(0, 1920)
    plt.ylim(0, 1080)
    plt.gca().invert_yaxis()
    plt.xlabel('X Position')
    plt.ylabel('Y Position')
    plt.title('Mouse Movement Trajectory')
    plt.legend()
    st.pyplot(plt)

```
