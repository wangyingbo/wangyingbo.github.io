---
layout: post
title: 利用Runtime给UIViewController的类别增加属性实现简单代码实现自定义导航栏左、右按钮
tags:  [Runtime]
categories: [iOS开发]
author: 王颖博
excerpt: "利用Runtime给UIViewController的类别增加属性-简单代码实现自定义导航栏左、右按钮"
---

- ***利用Runtime给UIViewController的类别增加属性-简单代码实现自定义导航栏左、右按钮***

- > 一、UIViewController的类别

由于做项目的时候，有时候需要自定义导航栏的leftBarButtonItem和rightBarButtonItem，并且由于有的页面的不同，造成返回按钮到左边边框的距离不同。所以每个新页面都要自定义返回按钮着实麻烦。于是就想着能不能给Controller抽出来一个类，用简单的几句代码实现自定义导航栏的返回按钮和右边的按钮。于是就有了下面的文章。

**1**
为了实现上面你的功能，首先需要给UIViewController建一个扩展类别。初始化方法UIViewController+Extension.h里：

    /**定义左边按钮block*/
    typedef void(^backButtonBlock)(UIButton *BackButton);
    /**定义右边按钮block*/
    typedef void(^rightButtonBlock)(UIButton *rightButton);

    /**
        *  添加左边返回按钮
        *
        *  @param buttonW        返回按钮宽
        *  @param buttonH        返回按钮高
        *  @param title          返回按钮的title
        *  @param titleColor     字体颜色
        *  @param buttonImage    北京图片
        *  @param negativeSpacer 间隔距离
        *  @param block          回调block
        */
    - (void)initBackButtonWithButtonW:(CGFloat)buttonW
                                buttonH:(CGFloat)buttonH
                                title:(NSString *)title
                            titleColor:(UIColor *)titleColor
                            buttonImage:(UIImage *)buttonImage
                        negativeSpacer:(CGFloat)negativeSpacer
                            touchBlock:(backButtonBlock)block;
    
    
    
    /**
        *  添加右边返回按钮
        *
        *  @param buttonW    右边按钮宽
        *  @param buttonH    右边按钮高
        *  @param title      右边按钮的title
        *  @param titleColor 字体颜色
        *  @param block      回调block
        */
    - (void)initRightButtonWithButtonW:(CGFloat)buttonW
                                buttonH:(CGFloat)buttonH
                                    title:(NSString *)title
                            titleColor:(UIColor *)titleColor
                            touchBlock:(rightButtonBlock)block;
                            
这里定义两个block，是为了把按钮传出去，给Controller可以修改button属性使用的。


**2**
接下来，要在UIViewController+Extension.m里实现方法，具体方法，略去，文末放上demo，可以查看。

**3**
然而，当实现完毕后，发现button的点击事件不能再类别里写，因此我们需要在类别里实现button的点击事件，并且能够把button的点击事件传出去，这样才符合设计精神。在这时，万能的block又立功了。我的想法是，再设计一个secondBlock，当button的点击事件发生时，在点击响应方法里发送secondBlock，这样在Controller里就能通过secondBlock的回调方法捕捉类别里的button的点击事件，就能完成任务。

    /**定义左边按钮点击事件block*/
    typedef void(^leftBtnTargetBlock)(NSString *string);
    /**定义右边按钮点击事件block*/
    typedef void(^rightBtnTargetBlock)(NSString *string);
    @interface UIViewController (Extension)
    
    @property (nonatomic, copy)   NSString *name;  //视图名字
    @property (nonatomic, assign) BOOL  hasChildViewController;//是否有子视图
    @property (nonatomic, strong) UIImage *backgroundImage;   //背景图片
    
    /**右边按钮点击事件block*/
    @property (nonatomic, copy) rightBtnTargetBlock rightBtnBlock;
    /**左边按钮点击事件block*/
    @property (nonatomic, copy) leftBtnTargetBlock leftBtnBlock;
    
    /**
     *  添加左边返回按钮
     *
     *  @param buttonW        返回按钮宽
     *  @param buttonH        返回按钮高
     *  @param title          返回按钮的title
     *  @param titleColor     字体颜色
     *  @param buttonImage    北京图片
     *  @param negativeSpacer 间隔距离
     *  @param block          回调block
     */
    - (void)initBackButtonWithButtonW:(CGFloat)buttonW
                              buttonH:(CGFloat)buttonH
                                title:(NSString *)title
                           titleColor:(UIColor *)titleColor
                          buttonImage:(UIImage *)buttonImage
                       negativeSpacer:(CGFloat)negativeSpacer
                           touchBlock:(backButtonBlock)block;
    
    /**
     *  添加右边返回按钮
     *
     *  @param buttonW    右边按钮宽
     *  @param buttonH    右边按钮高
     *  @param title      右边按钮的title
     *  @param titleColor 字体颜色
     *  @param block      回调block
     */
    - (void)initRightButtonWithButtonW:(CGFloat)buttonW
                               buttonH:(CGFloat)buttonH
                                 title:(NSString *)title
                            titleColor:(UIColor *)titleColor
                            touchBlock:(rightButtonBlock)block;
    
    /**
     *  触摸屏幕隐藏键盘
     */
    - (void)setUpForDismissKeyboard:(UIView *)selfView;
    
    @end


然而，当我给UIController的类别添加属性时记起来了，类别一般情况下是不能添加属性的。那怎么办呢？别怕，我们还有Runtime的黑魔法。

                            
- > 二、利用Runtime给UIViewController的类别增加属性

Runtime是运行时机制，具体的原理这里不用探讨，我们只探讨怎么利用Runtime来给类别添加属性。

**1**
在步骤一里我们已经给类别添加了block属性和其他属性，因为类别里不能生成setter和getter方法，因此我们需要在.m文件里手动写这两个方法。

**2**
首先要分配内存地址，我们可以在.m文件里这样写
        
    #import <objc/runtime.h>
    static const void *kName = "name";
    static const void *kHasChildViewController = @"hasChildViewController";
    static const void *kBackgroundImage = @"backgroundImage";
    
    static const void *leftBlock = &leftBlock;
    static const void *rightBlock = &rightBlock;
    
内存是通过静态常量直接分配内存的。
    
**3**
接下来，利用AssociatedObject具体实现

    @implementation UIViewController (Extension)
    @dynamic leftBtnBlock;
    @dynamic rightBtnBlock;
    
    #pragma mark - 左边按钮点击事件block绑定
    - (leftBtnTargetBlock)leftBtnBlock
    {
        return objc_getAssociatedObject(self, leftBlock);
    }
    
    - (void)setLeftBtnBlock:(leftBtnTargetBlock)leftBtnBlock
    {
        objc_setAssociatedObject(self, leftBlock, leftBtnBlock, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    
    #pragma mark - 右边按钮点击事件block绑定
    - (rightBtnTargetBlock)rightBtnBlock
    {
        return objc_getAssociatedObject(self, rightBlock);
    }
    
    - (void)setRightBtnBlock:(rightBtnTargetBlock)rightBtnBlock
    {
        objc_setAssociatedObject(self, rightBlock, rightBtnBlock, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    
    #pragma mark - 字符串类型的动态绑定
    - (NSString *)name
    {
        return objc_getAssociatedObject(self, kName);
    }
    
    - (void)setName:(NSString *)name
    {
        objc_setAssociatedObject(self, kName, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
    }
    
    #pragma mark - BOOL类型的动态绑定
    - (BOOL)hasChildViewController
    {
        return [objc_getAssociatedObject(self, kHasChildViewController) boolValue];
    }
    
    - (void)setHasChildViewController:(BOOL)hasChildViewController
    {
        objc_setAssociatedObject(self, kHasChildViewController, [NSNumber numberWithBool:hasChildViewController], OBJC_ASSOCIATION_ASSIGN);
    }
    
    #pragma mark - 类类型的动态绑定
    - (UIImage *)backgroundImage
    {
        return objc_getAssociatedObject(self, kBackgroundImage);
    }
    
    - (void)setBackgroundImage:(UIImage *)backgroundImage
    {
        objc_setAssociatedObject(self, kBackgroundImage, backgroundImage, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }

- > 三、可以这样在Controller里调用

        /**
         *  添加左按钮
         */
        - (void)addLeftButton
        {
            [self initBackButtonWithButtonW:40 buttonH:30 title:@"返回" titleColor:[UIColor orangeColor] buttonImage:nil negativeSpacer:1 touchBlock:^(UIButton *BackButton) {
            }];
            WS(weakSelf)
            weakSelf.leftBtnBlock = ^(NSString *string){
                [weakSelf.navigationController popViewControllerAnimated:YES];
            };
        }
     
        /**
         *  添加右按钮
         */
        - (void)addRightButton
        {
            [self initRightButtonWithButtonW:40 buttonH:30 title:@"保存" titleColor:[UIColor greenColor] touchBlock:^(UIButton *rightButton) {
            }];
            WS(weakSelf)
            weakSelf.rightBtnBlock = ^(NSString *rightString){
                SHOW_ALTER(@"点击了右键");
            };
        }

- > 四、项目demo参看这里[DEMO](https://github.com/wangyingbo/UIViewControllerCategoryWithRuntime)

