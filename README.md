# python
好玩的万花尺项目

import turtle
import random
import math
from fractions import gcd

class Spiro:
    def __init__(self, xc, yc, col, R, r, l):
        #设置画笔的实例对象
        self.t = turtle.Turtle()
        #设置画笔的形状，其他的如“arrow”
        self.t.shape("turtle")
        #设置角度增量
        self.step = 5
        #定义一个bool变量，记住当前状态
        self.drawingComplete = False
        #定义一个方法，使其帮助初始化
        self.setParams(xc, yc, col, R, r, l)
        #定义一个方法，用于重启
        self.restart()

    def setParams(self, xc, yc, col, R, r, l):
        #设置各参数
        self.xc = xc
        self.yc = yc
        self.col = col
        self.R = int(R)
        self.r = int(r)
        self.l = l
        #gcd方法用于找出两个数的最大公约数
        gcdVal = gcd(self.r, self.R)
        #得到画完时小圆圈转动的圈数
        self.nRot = self.r // gcdVal
        self.k = r / R
        #设置颜色
        self.t.color(*col)
        #设置初始的角度
        self.a = 0
        
    def restart(self):
        #将bool变量设为FALSE
        self.drawingComplete = False
        #显示画笔
        self.t.showturtle()
        #抬起画笔
        self.t.up()
        #调用之前的参数，是程序看起来紧凑
        R, k, l = self.R, self.k, self.l
        a = 0
        #得到初始位置
        x = R * ((1 - k) * math.cos(a) + l * k * math.cos((1 - k) * a / k))
        y = R * ((1 - k) * math.sin(a) - l * k * math.sin((1 - k) * a / k))
        self.t.setpos(self.xc + x, self.yc + y)
        #放下画笔，接下来要画画了
        self.t.down()

    def draw(self):
        #设置好画笔的速度
        self.t.speed("normal")
        R, k, l = self.R, self.k, self.l
        #通过循环得到每次画笔到达的点坐标，再将各个点依次连接起来
        for i in range(0, 360 * self.nRot + 1, self.step):
            a = math.radians(i)
            x = R * ((1 - k) * math.cos(a) + l * k * math.cos((1 - k) * a / k))
            y = R * ((1 - k) * math.sin(a) - l * k * math.sin((1 - k) * a / k))
            self.t.setpos(self.xc + x, self.yc + y)
        #画完后，隐藏画笔，留下图形，如果图像会消失，在后面加个turtle.done()即可，如下注释
        self.t.hideturtle()
        #turtle.done()
            
    def update(self):
        #判断是否处于画完的状态
        if self.drawingComplete:
            return
        R, k, l = self.R, self.k, self.l
        #计算出当前的角度
        self.a += self.step
        #将角度转换成弧度数
        a = math.radians(self.a)
        #计算出更新后（一个增量step的弧度变化）所在点，并将画笔移动到那个位置
        x = R * ((1 - k) * math.cos(a) + l * k * math.cos((1 - k) * a / k))
        y = R * ((1 - k) * math.sin(a) - l * k * math.sin((1 - k) * a / k))
        self.t.setpos(self.xc + x, self.yc + y)
        #判断此时是否画完，画完的话将bool变量置为True，隐藏画笔
        if self.a >= 360 * self.nRot:
            self.drawingComplete = True
            self.t.hideturtle()
        
    def clear(self):
        #清除所有的痕迹
        self.t.clear()

#这个类用来得到随机的参数，即得到随机的螺旋线
class SpiroAnimator:
    def __init__(self, N):
        #设置时钟周期，单位是微秒
        self.deltaT = 10
        #设置画布的宽和高
        self.width = turtle.window_width()
        self.height = turtle.window_height()
        #定义一个名为spiros的空列表，用来存放不同的spiro实例对象
        self.spiros = []
        #得到N个参数是随机值的spiro对象
        for i in range(N):
            #设置参数为随机值
            rparams = self.getRandomParams()
            #得到参数为随机值的实例对象，参数是元组类型的
            spiro = Spiro(*rparams)
            #将这些对象添加到spiros列表中
            self.spiros.append(spiro)
        #设置更新频率，没delatT时长update一次
        turtle.ontimer(self.update, self.deltaT)
            
    def getRandomParams(self):
        width, height = self.width, self.height
        #得到设置范围内的不同随机值R, r, l, xc, yc, col
        R = random.randint(50, min(width, height) // 2)
        #randint左右均可取等号，闭区间
        r = random.randint(10, 9 * R // 10)
        #得到浮点数
        l = random.uniform(0.1, 0.9)
        xc = random.randint(- width // 2, width // 2)
        yc = random.randint(- height // 2, heigt // 2)
        #此处是得到0-1范围内的值，颜色的设置为（a, b, c），均为0-1内的浮点数
        col = (
            random.random(),
            random.random(),
            random.random()
        )
        return (xc, yc, col, R, r, l)
        
    def update(self):
        #设置一个变量，用来记住bool变量是否为True
        count = 0
        #调用spiro的更新方法，若其已画完，count加一
        for spiro in self.spiros:
            spiro.update()
            if spiro.drawingComplete:
                count += 1
        #若所有的都画完了，重启
        if count == len(self.spiros):
            self.restart()
        #不然的话，按设置继续更新
        turte.ontimer(self.update, self.deltaT)
    
    def restart(self):
        #清除每一个实例对象内的全部内容
        for spiro in self.spiros:
            spiro.clear()
            #得到随机参数，将其赋值给spiro实例对象，完成初始化条件然后重启
            rparams = self.getRandomParams()
            spiro.setParams(*rparams)
            spiro.restart()

    #定义一个函数，用来随时去掉画笔
    def toggleTurtles(self):
        for spiro in self.spiros:
            if spiro.t.isvisible():
                spiro.t.hideturtle()
            else:
                spiro.t.showturtle()

#根据自己的需求，设置不同的实例对象或进行不同的测试
if __name__ == "__main__":
    s = Spiro(0, 0, (0.2, 0.6, 0.9), 200, 5, 0.9)

“”“    
关于fractions模块的其他用法：

（方法很简单，就是得到不同情况下的最简分数而已）
>>> from fractions import Fraction

>>> Fraction(16, -10)  #分子分母

Fraction(-8, 5)

>>> Fraction(123)   #分子

Fraction(123, 1) 

>>> Fraction('3/7')   #字符串分数

Fraction(3, 7) 

>>> Fraction('-.125')  #字符串浮点数

Fraction(-1, 8) 

>>> Fraction(2.25)  #浮点数

Fraction(9, 4) 

>>> from decimal import Decimal

>>> Fraction(Decimal('1.1')) #Decimal

Fraction(11, 10)

>>> from fractions import Fraction

>>> a = Fraction(1,2)

>>> a

Fraction(1, 2)

>>> b = Fraction('1/3')

>>> b

Fraction(1, 3)

>>> a + b

Fraction(5, 6)

>>> a - b

Fraction(1, 6)
“”“
