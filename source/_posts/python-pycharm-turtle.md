---
title: pycharm不识别turtle的解决方法
date: 2020-05-14 11:49:57
categories:
  - python
tags:
  - python
---

pycharm不识别turtle的解决方法
<!--more-->

## 原因

pycharm和turtle库有冲突，不能自己识别出turtle下的方法

## 解决

找到turtle.py， 对turtle库的源码进行如下修改： 
注释掉原来的_ all_,新增如下
```
# __all__ = (_tg_classes + _tg_screen_functions + _tg_turtle_functions +
#            _tg_utilities + ['Terminator']) # + _math_functions)
__all__ = ['ScrolledCanvas', 'TurtleScreen', 'Screen', 'RawTurtle', 'Turtle', 'RawPen', 'Pen', 'Shape', 'Vec2D', 'back','backward', 'begin_fill', 'begin_poly', 'bk', 'addshape', 'bgcolor', 'bgpic', 'bye', 'clearscreen','colormode', 'delay', 'exitonclick', 'getcanvas', 'getshapes', 'listen', 'mainloop', 'mode', 'numinput','onkey', 'onkeypress', 'onkeyrelease', 'onscreenclick', 'ontimer', 'register_shape', 'resetscreen','screensize', 'setup', 'Terminator', 'setworldcoordinates', 'textinput', 'title', 'tracer', 'turtles','update', 'window_height', 'window_width', 'write_docstringdict', 'done', 'circle', 'clear', 'clearstamp','clearstamps', 'clone', 'color', 'degrees', 'distance', 'dot', 'down', 'end_fill', 'end_poly', 'fd','fillcolor', 'filling', 'forward', 'get_poly', 'getpen', 'getscreen', 'get_shapepoly', 'getturtle', 'goto','heading', 'hideturtle', 'home', 'ht', 'isdown', 'isvisible', 'left', 'lt', 'onclick', 'ondrag', 'onrelease','pd', 'pen', 'pencolor', 'pendown', 'pensize', 'penup', 'pos', 'position', 'pu', 'radians', 'right', 'reset','resizemode', 'rt', 'seth', 'setheading', 'setpos', 'setposition', 'settiltangle', 'setundobuffer', 'setx','sety', 'shape', 'shapesize', 'shapetransform', 'shearfactor', 'showturtle', 'speed', 'st', 'stamp', 'tilt', 'tiltangle', 'towards', 'turtlesize', 'undo', 'undobufferentries', 'up', 'width', 'write', 'xcor', 'ycor']
```
