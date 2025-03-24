---
layout: post
title: "CMU 15112 Term Project"
subtitle: "SpongeBob Themed Game in Python"
author: "Yingqian Cao"
header-style: text
tags:
  - CMU
  - Python
---

Introduction
--

This article shares my term project for [Carnegie Mellon University's 15112 course (Fundamentals of Programming)](https://www.cs.cmu.edu/~112/syllabus.html) - [a SpongeBob-themed 2D platform game developed in Python](https://www.bilibili.com/video/BV1HG4y1V755/?spm_id_from=333.1387.homepage.video_card.click&vd_source=1502ea6f8fd7ecde5aff19ec5001af2d). The project utilizes the `cmu_112_graphics` library provided by the course to simplify graphics and animation handling. This article demonstrates how I built a simple game from scratch, including game design, code implementation, and key programming techniques.
![](https://github.com/YC-Andrea/yc-andrea.github.io/blob/master/img/Screen%20Shot%202025-03-24%20at%209.59.53%20PM.png)


Project Background
--

This project was the term assignment for CMU 15112, designed to reinforce programming concepts learned in class through practical application. The requirements were to design and implement a moderately complex game, and I chose to develop a 2D platformer.


Game Design
--

Inspired by classic platformer games, players control SpongeBob to move across platforms, collect Krabby Patties, and avoid Plankton's pursuit. The game features two levels with different layouts and challenges. Players can control SpongeBob's movement using keyboard inputs and use weapons to defeat Plankton.



Tech Stack
--

* Programming Language: Python
* Graphics Library: `cmu_112_graphics` (simplified graphics library provided by CMU 15112)
* Resource Management: Loading and using image/sound files
* Game Logic: Collision detection, character movement, level design



Code Structure
--

1. Game Initialization
The `appStarted` function initializes game resources including images, sounds, character positions, and platform layouts:
```ts
def appStarted(app):
    app.sound_win = Sound('win.mp3')
    app.sound_lose = Sound('lose.mp3')
    app.sound_hit = Sound('hit.mp3')
    app.sound = Sound('music.mp3')
    app.image_url = app.loadImage('background.png')
    app.image_SBSP = app.loadImage('SBSP.png')
    app.image_KB = app.loadImage('KB.png')
    app.image_platform = app.loadImage('platform.png')
    app.image_ladd = app.loadImage('ladder2.png')
    app.sprites = [app.loadImage(f'P{i}.png') for i in range(5)]
    app.spriteCounter = 0
```

2. Game Logic
The keyPressed and timerFired functions handle player input and game state updates:
```ts
def keyPressed(app, event):
    if event.key == 'Up':
        app.SBSP_y1 -= app.SBSP_v1
    elif event.key == 'Down':
        app.SBSP_y1 += app.SBSP_v1
    elif event.key == 'Left':
        app.SBSP_x1 -= app.SBSP_v1
    elif event.key == 'Right':
        app.SBSP_x1 += app.SBSP_v1
    elif event.key == 'Space':
        if ShotPlankton(app, app.SBSP_x1, app.SBSP_y1, app.P1_x, app.P1_y):
            app.weapon -= 1
            app.lives += 1
```

3. Game Rendering
The redrawAll function draws all game elements:
```ts
def redrawAll(app, canvas):
    if app.screencount == 0:
        DrawStartScreen(app, canvas)
    elif app.screencount == 1:
        DrawLevel1(app, canvas)
    elif app.screencount == 2:
        DrawLevel2(app, canvas)
    elif app.screencount == 4:
        DrawLose(app, canvas)
    elif app.screencount == 5:
        DrawWin(app, canvas)
```

4. Game Over
Win/lose screens are displayed when appropriate:
```ts
def DrawLose(app, canvas):
    canvas.create_image(500, 300, image=ImageTk.PhotoImage(app.image_lose2))
    canvas.create_text(app.width//2, app.height//9, 
                      text="Press 'r' to restart", font='Arial 30 bold')

def DrawWin(app, canvas):
    canvas.create_image(500, 300, image=ImageTk.PhotoImage(app.image_win2))
    canvas.create_text(app.width//2, app.height//9, 
                      text="You Win!", font='Arial 30 bold')
```



Challenges and Solutions
--

1.Collision Detection: Implemented efficient collision logic using geometric calculations
2.Resource Management: Carefully managed loading/releasing of multiple image/sound assets
3.Level Design: Balanced difficulty and fun factor across different level layouts

Conclusion
--

This project not only reinforced my programming knowledge from CMU 15112 but also deepened my understanding of key game development concepts like resource management, input handling, collision detection, and game state management. I hope this article inspires you to begin your own game development journey!

Feel free to leave comments or questions below!


参考
--

1.  JavaScript 模块化七日谈 - 黄玄的博客 Hux Blog [https://huangxuan.me/2015/07/09/js-module-7day/](https://huangxuan.me/2015/07/09/js-module-7day/)
2.  如何理解尤雨溪在 2019 VueConf 上所讲的 UI 类框架很少使用面向对象的特性这件事？- 黄玄的回答 [https://www.zhihu.com/question/328958700/answer/714287394](https://www.zhihu.com/question/328958700/answer/714287394)
3.  前端是否适合使用面向对象的方式编程？- 黄玄的回答 [https://www.zhihu.com/question/329005869/answer/739525268](https://www.zhihu.com/question/329005869/answer/739525268)
4.  React Hooks的引入会对之后的React项目开发产生什么影响？- 黄玄的回答 [https://www.zhihu.com/question/302916879/answer/536846510](https://www.zhihu.com/question/302916879/answer/536846510)
5.  React 上下文的组合是通过调用顺序在运行时里维护一个链表而非基于参数化多态的层叠（比如 Monad Transformer）来表达，可以看到都是线性的。
6.  State-passing Style Hooks [https://mobile.twitter.com/acdlite/status/971598256454098944](https://mobile.twitter.com/acdlite/status/971598256454098944)
7.  [https://github.com/reactjs/react-basic](https://github.com/reactjs/react-basic)
8.  可以把上下文或者 Hooks 的调用视为一次 stack unwinding + resume continuation。同样，考虑 row polymorphism 也是线性的。
9.  Algebraic Effects for the Rest of Us [https://overreacted.io/algebraic-effects-for-the-rest-of-us/](https://overreacted.io/algebraic-effects-for-the-rest-of-us/)
10.  通俗易懂的代数效应 [https://overreacted.io/zh-hans/algebraic-effects-for-the-rest-of-us/](https://overreacted.io/zh-hans/algebraic-effects-for-the-rest-of-us/)
11.  Why React [https://gist.github.com/sebmarkbage/a5ef436427437a98408672108df01919](https://gist.github.com/sebmarkbage/a5ef436427437a98408672108df01919)
12.  [https://swiftwithmajid.com/2019/06/12/understanding-property-wrappers-in-swiftui/](https://swiftwithmajid.com/2019/06/12/understanding-property-wrappers-in-swiftui/)
13.  [https://developer.android.com/jetpack/compose/state#remember](https://developer.android.com/jetpack/compose/state#remember)
