---
layout: post
title: 实现全屏模态半透明页面的效果和毛玻璃效果
tags:  [iOS]
categories: [iOS开发]
author: 王颖博
excerpt: "实现全屏模态半透明页面的效果和毛玻璃效果"
---

- ***实现全屏模态半透明页面的效果和毛玻璃效果***

- 这阵子产品里有一个效果，当mainVC模态跳转也即present一个testVC时，要让testVC的背景半透明，可以模糊看得到mainVC的一些东西。要求如下：![](https://github.com/wangyingbo/testPresentViewController/blob/master/images/1.jpg)
原以为是简单不过的事情，随手写了以下代码：

        mainVC代码：
        firstVC *first = [[firstVC alloc]init];
            [self presentViewController:first animated:YES completion:nil];
            
        testVC代码：
            self.view.backgroundColor = [UIColor blackColor];
            self.view.alpha = 0.7;
结果在present时发现了问题，当present到testVC时，发现testVC页面先是半透明了一下，然后很快就有变黑了。效果是这样的：

![](https://raw.githubusercontent.com/wangyingbo/testPresentViewController/master/images/gif.gif)

查了资料发现解释如下：

原因是如果新的ViewController以全屏的方式，完全盖住了原来的ViewController，那么ios为了节省内存，会自动将原来的ViewController的view给unload掉，所以背景就变黑了：
The “problem” is that iOS is very finicky about not wasting memory,and since the modal view will completely cover the one beneath it,it doesn’t make much sense to keep it loaded.Therefore, iOS unloads the view that presents the modal one.You may check this behavior by implementing -viewWillDisappear: and -viewDidDisappear:transparent viewcontroller
所以，用全屏的方式可能实现不了，或许需要override原来的ViewController的viewWillDisappear:方法.


- 后来找到了解决方法，如下：

    
        mainVC代码：
        firstVC *first = [[firstVC alloc]init];
        first.modalPresentationStyle = UIModalPresentationOverFullScreen;
        [self presentViewController:first animated:YES completion:nil];
        
        testVC代码：
        self.view.backgroundColor = [UIColor blackColor];
        self.view.alpha = 0.7;
        
        
![半透明](https://raw.githubusercontent.com/wangyingbo/testPresentViewController/master/images/gif1.gif)
        
- 还有一种毛玻璃效果       

        mainVC代码：
        secondVC *second = [[secondVC alloc]init];
        second.modalPresentationStyle = UIModalPresentationOverFullScreen;
        //modalTransitionStyle不可与vc.view.backgroundColor = [UIColor blackColor]同时使用
        [second setModalTransitionStyle:UIModalTransitionStyleCrossDissolve];
        [self presentViewController:second animated:YES completion:nil];
        testVC代码：
        //毛玻璃
        UIBlurEffect *blurEffect=[UIBlurEffect effectWithStyle:UIBlurEffectStyleDark];
        UIVisualEffectView *visualEffectView=[[UIVisualEffectView alloc]initWithEffect:blurEffect];
        [visualEffectView setFrame:self.view.bounds];
        [self.view addSubview:visualEffectView];
        
        
![毛玻璃](https://raw.githubusercontent.com/wangyingbo/testPresentViewController/master/images/gif2.gif)

- 另外，网上有一张方法，不过我试了一下发现并没有效果。大家可以试试。present一个窗口化的ViewController。但是这个窗口默认的背景色是磨砂不透明的，因此还需要把它的背景色设为透明。这样看起来就像是全屏遮罩一样，但是由于系统不认为新的View是全屏的，所以上一个View也不会被unload。如下：

        YLSLockScreenViewController *lockScreenController = [[YLSLockScreenViewController alloc] init];  
        lockScreenController.modalPresentationStyle = UIModalPresentationFormSheet;// 窗口  
        [self.mainViewController presentViewController:lockScreenController animated:YES completion:^(void){  
        lockScreenController.view.superview.backgroundColor = [UIColor clearColor];// 背景色透明  
        }];  
代码比较简单，需要注意的是，设置背景色透明的那行代码，需要写在completion block里，而且设置的不是controller.view.backgroundColor，而是controller.view.superview.backgroundColor。


- > 最后，附上代码:[demo](https://github.com/wangyingbo/testPresentViewController)。欢迎交流。