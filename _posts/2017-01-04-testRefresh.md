---
layout: post
title: 下拉刷新和上拉加载更多的原理
tags:  [iOS]
categories: [iOS开发]
author: 王颖博
excerpt: "下拉刷新和上拉加载更多的原理"
---



### *** 下拉刷新和上拉加载更多的原理***


### 一、介绍

在做App开发时,很多时候会用到下拉刷新和上拉加载,比如我比较常用第三方MJRefresh来实现,在这里主要讨论这种效果实现的原理




# pragma mark - UIScrollViewDelegate
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
	{
        //NSLog(@"偏移量:%f",scrollView.contentOffset.y);
        //NSLog(@"contentSize的        高:%f",scrollView.contentSize.height);
}
// 下拉刷新的原理
- (void)scrollViewWillBeginDecelerating:(UIScrollView *)scrollView
	{
        if (scrollView.contentOffset.y \< - 100) {
        
            [UIView animateWithDuration:DELAY_TIME animations:^{
]()            
                //  frame发生偏移,距离顶部150的距离(可自行设定)
                self.tableView.contentInset = UIEdgeInsetsMake(150.0f, 0.0f, 0.0f, 0.0f);
                NSLog(@"开始下拉刷新");
            } completion:^(BOOL finished) {
            
                /**                  *  发起网络请求,请求刷新数据
                 */
                dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(DELAY_TIME * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                    NSLog(@"下拉刷新结束了");
                    [UIView animateWithDuration:DELAY_TIME animations:^{
]()                    
                        self.tableView.contentInset = UIEdgeInsetsMake(0.0f, 0.0f, 0.0f, 0.0f);
                    }];
                });
            
            }];
        }
}

// 上拉加载的原理
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView     willDecelerate:(BOOL)decelerate
	{
	    
        NSLog(@"%f",scrollView.contentOffset.y);
        NSLog(@"%f",scrollView.frame.size.height);
        NSLog(@"%f",scrollView.contentSize.height);
        /**          *  关键--\>
         *  scrollView一开始并不存在偏移量,但是会设定contentSize的大小,所以contentSize.height永远都会比contentOffset.y高一个手机屏幕的
     *  高度;上拉加载的效果就是每次滑动到底部时,再往上拉的时候请求更多,那个时候产生的偏移量,就能让contentOffset.y + 手机屏幕尺寸高大于这
         *  个滚动视图的contentSize.height
         */
        if (scrollView.contentOffset.y + scrollView.frame.size.height \>= scrollView.contentSize.height) {
        
            NSLog(@"%d %s",__LINE__,__FUNCTION__);
            [UIView commitAnimations]();
        
            [UIView animateWithDuration:DELAY_TIME animations:^{
]()                //  frame发生的偏移量,距离底部往上提高60(可自行设定)
                self.tableView.contentInset = UIEdgeInsetsMake(0, 0, 60, 0);
                NSLog(@"上拉加载更多开始了");
            } completion:^(BOOL finished) {
            
                /**                  *  发起网络请求,请求加载更多数据
                 *  然后在数据请求回来的时候,将contentInset改为(0,0,0,0)
                 */
                dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(DELAY_TIME * NSEC_PER_SEC)),     dispatch_get_main_queue(), ^{
                    NSLog(@"上拉加载更多结束了");
                    [UIView animateWithDuration:DELAY_TIME animations:^{
]()                        self.tableView.contentInset = UIEdgeInsetsMake(0, 0, 0, 0);
                    }];
                
                });
            }];
        
        }
}

