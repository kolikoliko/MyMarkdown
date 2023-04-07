# python的学习



#### 八皇后

```python
def conflict(state, nextColumn):
    nextRow = rows = len(state)
    for row in range(rows):
        column = state[row]
        if abs(column - nextColumn) in [0, nextRow - row]:
            return True
    return False


def queens(num, state=()):
    for pos in range(num):
        if not conflict(state, pos):

            if len(state) == num - 1:
                yield pos,
            else:
                for result in queens(num, state + (pos,)):
                    yield (pos,) + result


if __name__ == '__main__':
    solutions = queens(8)
    for index, solution in enumerate(solutions):
        print('第%d种：' % (index + 1), solution, "\r\n")
```



#### 画菱形

```python
n = int(input('num:'))
for i in range(1, n+1, 1):
    print(('* ' * i).center(5*n))

for i in range(1, n, 1):
    print(('* ' * (n-i)).center(5*n))
```



#### 密码强度检测

```python
import string


def check(pwd):
    # 密码必须至少包含6个字符
    if len(pwd) < 6:
        return '密码少于六位字符'

    count = 0
    flag = [0, 0, 0, 0]
    # 分别用来标记pwd是否含有数字、小写字母、大写字母和指定的标点符号
    for ch in pwd:
        # 是否包含数字
        if ch in string.digits and flag[0] == 0:
            count += 1
            flag[0] = 1
        # 是否包含小写字母
        elif ch in string.ascii_lowercase and flag[1] == 0:
            count += 1
            flag[1] = 1
        # 是否包含大写字母
        elif ch in string.ascii_uppercase and flag[2] == 0:
            count += 1
            flag[2] = 1
        # 是否包含指定的标点符号
        elif ch in ',.!;?<>' and flag[3] == 0:
            count += 1
            flag[3] = 1
    # 统计包含的字符种类，返回密码强度

    if count == 1:
        return '弱密码'
    elif count == 2:
        return '中低强度'
    elif count == 3:
        return '中高强度'
    elif count == 4:
        return '强密码'
    else:
        return '密码出错'


password = input("请输入密码：")
print(check(password))
```



#### 小游戏

```python
import random
import time

n = ''
while n != 'n':
    win_flag = 2
    for c in range(3):
        b = c + 1
        print(('现在是第%d局' % b).center(37, '-'))
        time.sleep(1.5)

        player_dict = {'health': random.randint(100, 150), 'power': random.randint(30, 50)}
        enemy_dict = {'health': random.randint(100, 150), 'power': random.randint(30, 50)}

        print('-' * 40)
        print('【玩家】\r\n血量：%d\r\n攻击：%d' % (player_dict['health'], player_dict['power']))
        print('-' * 40)
        print('【敌人】\r\n血量：%d\r\n攻击：%d' % (enemy_dict['health'], enemy_dict['power']))

        time.sleep(1.5)

        while player_dict['health'] > 0 and enemy_dict['health'] > 0:
            print('-' * 40)
            enemy_dict['health'] -= player_dict['power']
            print("你发起了攻击，【敌人】剩余血量%d" % enemy_dict['health'])
            time.sleep(1.5)
            if enemy_dict['health'] <= 0:
                print('-' * 40)
                print('敌人死翘翘了，你赢了！')
                win_flag += 1
                break

            player_dict['health'] -= enemy_dict['power']
            print("敌人向你发起了攻击，【玩家】的血量剩余%d" % player_dict['health'])
            time.sleep(1.5)
            if player_dict['health'] <= 0:
                print('-' * 40)
                print('你死翘翘了，敌人赢了！')
                win_flag -= 1
                break
    time.sleep(1.5)
    player_dict = {'0': '你赢了！', '1': '你输了！'}
    result = ''
    if win_flag > 2:
        result = '0'
    elif win_flag < 2:
        result = '1'
    print("【最终结果】:%s" % player_dict[result])
    n = input("要继续游戏吗，请输入n退出，输入其他继续：")

```



#### 过河卒(DP算法)

```python 
import numpy as np

count = [0]
go_item = [[0, 0], [1, 2], [1, -2], [2, 1], [2, -1], [-1, 2], [-2, 1], [-2, -1], [-1, -2]]


def getBoard(hx, hy, boardX, boardY, notpassboard):
    for item in go_item:
        tx = hx + item[0]
        ty = hy + item[1]
        if 0 <= tx <= boardX and 0 <= ty <= boardY:
            notpassboard[tx, ty] = 1
    return notpassboard


def drawBoard(notpass):
    for pos in notpass:
        print(pos)


def dp(x, y, dx, dy, notPass, dpbook):
    for i in range(dx + 1):
        for j in range(dy + 1):
            if i == 0 and j == 0:
                continue
            if notPass[i][j] == 0:
                if i == 0:
                    ta = 0
                else:
                    ta = dpbook[i - 1][j]
                if j == 0:
                    tb = 0
                else:
                    tb = dpbook[i][j - 1]
                dpbook[i][j] = ta + tb
    return dpbook[dx][dy]


if __name__ == '__main__':
    bx, by, mx, my = [int(i) for i in input('请输入板子x,y参数，还有马的坐标x,y，利用空格隔开四个参数:').split(" ")]
    boardStop = np.zeros((bx + 1, by + 1), dtype=int)  # 生成m * n的全0数组
    boardCount = np.zeros((bx + 1, by + 1), dtype=int)  # 生成m * n的全0数组
    boardCount[0][0] = 1
    boardStop = getBoard(mx, my, bx, by, boardStop)
    drawBoard(boardStop)
    methods = dp(0, 0, bx, by, boardStop, boardCount)
    print(methods)

```

