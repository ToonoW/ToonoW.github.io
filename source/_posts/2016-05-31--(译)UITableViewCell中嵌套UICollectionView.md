---
date: 2016-05-31
layout: post
title: (译)UITableViewCell中嵌套UICollectionView
category: iOS
---

## 说在前面

在写的一个App用到了`UITableViewCell`中嵌套`UICollectionView`的设计，原以为会很简单，但是后来发现还是问题多多的，例如`DataSource`的设置问题，还有我对重用中的一些理解的混乱，巴拉巴拉巴拉。所以就找到了篇的原文。  

原文：[Putting a UICollectionView in a UITableViewCell](http://ashfurrow.com/blog/putting-a-uicollectionview-in-a-uitableviewcell/)

  

## 正文

所以，你希望将collection view放进一个table view cell里面是吧？这听起来很简单对吧？好，做好这件简单的事情需要一点点准备工作。我们希望每个类结构清晰各司其职，那么`UITableViewCell`不能充当`UICollectionView`的`DataSource`和`delegate`（因为不分开的话会非常糟糕）。你可以下载[这里](https://github.com/AshFurrow/AFTabledCollectionView)的示例代码。  

   

我们打算构建一个视图，结构如下图。每一个`UITableViewCell`会嵌套一个`UICollectionView`。(为了某些原因，我们到的collection view 会是自定义的子类。)每一个collection view 装着确定数目的cell，cell的数目由`DataSource`给出。  

![view的结构图](http://ashfurrow.com/img/import/blog/putting-a-uicollectionview-in-a-uitableviewcell/AFE11F3C86B04CDF9EDB1F080C6668EB.png)

  

每一个 collection view与table view有同样的`Datasource`和`delegate`，自定义`UICollectionView`的子类也是我们需要做的。  

![diagram of DataSource&delegate](http://ashfurrow.com/img/import/blog/putting-a-uicollectionview-in-a-uitableviewcell/E26436B73EEE4D06A38646AEDAFC9692.png)         

 上面的图示向你展示了这个demo较为复杂的部分，如何将view controller看成`UICollectionView`的`delegate`。我们上面说到的子类继承了`UICollectionView`，我们给它一个`index`属性（`property`）。一半情况下，继承`UICollectionView`是比较少见的，不过在这里还是可以接受这种做法的。  

  

将collection view 加到cell里面还是太早了，我们还需要做些工作。我们在cell 特定的初始化器中用标准的`UICollectionViewFlowLayout`实例化`AFCollectionView`的对象。  

```objective-c
- (id)initWithStyle:(UITableViewCellStyle)style 
    reuseIdentifier:(NSString *)reuseIdentifier
{
    if (!(self = [super initWithStyle:style reuseIdentifier:reuseIdentifier])) return nil;

    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
    layout.sectionInset = UIEdgeInsetsMake(10, 10, 10, 10);
    layout.itemSize = CGSizeMake(44, 44);
    layout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
    self.collectionView = [[AFIndexedCollectionView alloc] initWithFrame:CGRectZero collectionViewLayout:layout];
    [self.collectionView registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:CollectionViewCellIdentifier];
    self.collectionView.backgroundColor = [UIColor whiteColor];
    self.collectionView.showsHorizontalScrollIndicator = NO;
    [self.contentView addSubview:self.collectionView];

    return self;
}
```

​    

我们在`layoutSubviews`里面调整collection view 的frame去适应并且填满cell。

```objective-c
-(void)layoutSubviews
{
    [super layoutSubviews];

    self.collectionView.frame = self.contentView.bounds;
}
```

  

接下来我们将在view colltroller里面设置我们的model。我们用多种`UIColor`来设置我们的模型，简单又明显。每一个cell将显示各不相同而且随机的颜色。   

  

这个model是为了反映每个table view 和每个collection view的效果。我们首先用一个数组存储多个对象，每个对象反映一个table cell。然后这些对象它们自己也是数组，每个数组里的对象反映的是collection view cell里面的内容。

```objective-c
-(NSInteger)tableView:(UITableView *)tableView 
    numberOfRowsInSection:(NSInteger)section
{
    return self.colorArray.count;
}
```

   

接下来我们去实现`UICollectionViewDataSource`的方法。

```objective-c
-(NSInteger)collectionView:(AFIndexedCollectionView *)collectionView 
    numberOfItemsInSection:(NSInteger)section
{
    NSArray *collectionViewArray = self.colorArray[collectionView.index];
    return collectionViewArray.count;
}

-(UICollectionViewCell *)collectionView:(AFIndexedCollectionView *)collectionView 
    cellForItemAtIndexPath:(NSIndexPath *)indexPath
{    
    UICollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:CollectionViewCellIdentifier forIndexPath:indexPath];

    NSArray *collectionViewArray = self.colorArray[collectionView.index];
    cell.backgroundColor = collectionViewArray[indexPath.item];

    return cell;
}
```

  

你会看到我们使用的`index`是collection view的属性，我们用它来查询到适合的model去配置我们的collection view cell。请注意，view controller不会存储任何指向collection view的引用，collection view们只属于`UITableViewCell`们。这是个好消息，因为它们都会被重用，这会节省我们的内存资源。  

  

但是我们从哪里得到`index`？这也是我们需要做的。  

```objective-c
-(void)tableView:(UITableView *)tableView willDisplayCell:(AFTableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath
{
    [cell setCollectionViewDataSourceDelegate:self index:indexPath.row];
}
```

​     

剩下要做的事情就是去“记住”每个collection view在其所在的table cell将要不显示（被滑动离开屏幕）时候的collection view内容的偏移量（collection view的content被滑动的距离），这样子就可以达到“记住”用户在不同table cell滑动collection view后的位置不在初始位置的效果了。我们这里利用`NSMutableDictionary`来记住我们内容的偏移量。  

```objective-c
-(void)tableView:(UITableView *)tableView 
    willDisplayCell:(AFTableViewCell *)cell 
    forRowAtIndexPath:(NSIndexPath *)indexPath
{
    [cell setCollectionViewDataSourceDelegate:self index:indexPath.row];
    NSInteger index = cell.collectionView.index;

    CGFloat horizontalOffset = [self.contentOffsetDictionary[[@(index) stringValue]] floatValue];
    [cell.collectionView setContentOffset:CGPointMake(horizontalOffset, 0)];
}

-(void)tableView:(UITableView *)tableView 
    didEndDisplayingCell:(AFTableViewCell *)cell 
    forRowAtIndexPath:(NSIndexPath *)indexPath
{
    CGFloat horizontalOffset = cell.collectionView.contentOffset.x;
    NSInteger index = cell.collectionView.index;
    self.contentOffsetDictionary[[@(index) stringValue]] = @(horizontalOffset);
}
```

![content offset](http://ashfurrow.com/img/import/blog/putting-a-uicollectionview-in-a-uitableviewcell/1DA58865F87F4E9696A16088F491E04D.png)

​    

以上就是我们全文的主要内容啦。不需要很多的代码，但是做好它需要在编码前进行稍微的设计结构。你可以在Github下载[完整的代码](https://github.com/AshFurrow/AFTabledCollectionView)。  

  

最后还有作者的安利：）。真心希望你喜欢我的教程，我还推荐一下我的电子书[`UICollectionView`: The Complete Guide](http://click.linksynergy.com/fs-bin/click?id=3JVIZPzOhac&subid=&offerid=145238.1&type=10&tmpid=3559&RD_PARM1=http%253A%252F%252Fwww.informit.com%252Fstore%252Fios-uicollectionview-the-complete-guide-9780133410945)。在书里面我会里里外外地向你介绍如何使用`UICollectionView`。你可以马上预定然后下载所以已经写出来的章节，心动不如行动。括弧笑。