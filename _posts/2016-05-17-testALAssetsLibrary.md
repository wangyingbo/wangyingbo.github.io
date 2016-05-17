---
layout: post
title: 根据<AssetsLibrary/AssetsLibrary.h>框架自定义相册
tags:  [iOS],[AssetsLibrary]
categories: [iOS开发]
author: 王颖博
excerpt: "根据<AssetsLibrary/AssetsLibrary.h>框架自定义相册"
---
- > **testALAssetsLibrary**
- iOS开发中有时候会经常需要拍照和选取图片，拍照直接调用UIImageViewPicker就可以了，如若要自定义相机的话可以自己定义拍照页面，此先略过不讲。这篇文章只讲述如何自定义相册——利用系统的<AssetsLibrary/AssetsLibrary.h>框架。
- 首先第一步，当然是要导入<AssetsLibrary/AssetsLibrary.h>框架 -


> >导入框架

    #import <AssetsLibrary/AssetsLibrary.h>
    #import <AssetsLibrary/ALAsset.h>
    #import <AssetsLibrary/ALAssetsLibrary.h>
    #import <AssetsLibrary/ALAssetsGroup.h>
    #import <AssetsLibrary/ALAssetRepresentation.h>
    
    
- >循环遍历ALAssetsLibrary，调用enumerateGroupsWithTypes的block。
    
        [_assetsLibrary enumerateGroupsWithTypes:ALAssetsGroupAll usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
        if (group)
        {
            //NSLog(@"*****相册个数***%@",self.groupMutArr);
            [self.groupMutArr addObject:group];
            //每个相册的名字
            NSString *groupName = [group valueForProperty:ALAssetsGroupPropertyName];
            [self.groupName addObject:groupName];
            
            for (ALAssetsGroup *_group in self.groupMutArr)
            {
                [_group enumerateAssetsUsingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
                    if (result)
                    {
                        [self.imageArr addObject:result];
                        //NSLog(@"*****所有相册里的所有图片****%@",self.imageArr);
                        //UIImage *image = [UIImage imageWithCGImage: result.thumbnail];
                        //NSString *type=[result valueForProperty:ALAssetPropertyType];
                    }
                }];
            }
        }
        
        [self.collectionView reloadData];
        
        } failureBlock:^(NSError *error) {
        NSLog(@"获取相册失败");
        }];

- >根据数据数组来定义collectionView，返回数组

        pragma mark - UICollectionViewDataSource
        - (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
        {
            NSMutableArray *testMutArr = [NSMutableArray array];
            ALAssetsGroup *testGroup = [self.groupMutArr objectAtIndex:section];
            [testGroup enumerateAssetsUsingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
                if (result)
                {
                    [testMutArr addObject:result];
                }
            }];
            
            return testMutArr.count;
        }
        
        - (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView
        {
            return self.groupMutArr.count;
        }
        
        - (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
        {
            YBTNFirstCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:TNFirstCell forIndexPath:indexPath];
            
            NSMutableArray *mutArr = [NSMutableArray array];
            ALAssetsGroup *testGroup = [self.groupMutArr objectAtIndex:indexPath.section];
            [testGroup enumerateAssetsUsingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
                if (result){
                    [mutArr addObject:result];
                }}];
            
            ALAsset *result = [mutArr objectAtIndex:indexPath.item];
            UIImage *image = [UIImage imageWithCGImage: result.thumbnail];
            [cell setCellWithImage:image];
            return cell;
        }
        
        
- > 项目截图如下：
![截图](https://raw.githubusercontent.com/wangyingbo/testALAssetsLibrary/master/image.png)

>参考在此[项目demo](https://github.com/wangyingbo/testALAssetsLibrary)
