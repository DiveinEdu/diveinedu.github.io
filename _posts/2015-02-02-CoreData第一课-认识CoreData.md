---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, Core Data, 数据存储, Mac, 应用开发]
---
####问题
在iOS／Mac中我们有许多方式去持久化存储数据：NSUserDefault、Key chain、C语言文件接口、NSFileHandle、基础框架中的write方法、归档等等。在实际应用中，我们经常需要将这些数据按一定格式转换为对象，并且进行一定的筛选等操作然后再使用，显得不是很方便。Apple给我们提供了Core Data框架，可以直接按对象的方式操作数据，让这些变得非常简单。

####简介
CoreData中有这么几个常用的元素：

|名称|作用|
|:-:|:-:|
|NSManagedObjectModel|对象模型，指定所用对象文件|
|NSPersistentStoreCoordinator|持久化存储协调器，设置对象的存储方式和数据存放位置|
|NSManagedObjectContext|对象管理上下文，负责数据的实际操作(重要)|
|NSEntityDescriptor|实体描述符，描述一个实体，可以用来生成实体对应的对象|
|NSManagedObject|对象|
|NSFetchRequest|对象查询，相当于SQL的Select语句|

![](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/CoreData/Art/coredata_doc_management_2x.png)

####使用步骤
在这篇文章中我们使用最简单的方式，也就是在创建项目的时候，勾选“Core Data”选项。Xcode会自动替我们在“AppDelegate”中加入创建“NSManagedObjectModel”、“NSPersistentStoreCoordinate”和“NSManagedObjectContext”等对象，方便后面的使用。

1 . 创建“NSManagedObjectModel”对象。

```objc
- (NSManagedObjectModel *)managedObjectModel {
    if (_managedObjectModel != nil) {
        return _managedObjectModel;
    }
    
    //CoreData模型文件的路径，注意编译好的模型文件名扩展名为"momd"
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"CoreData01" withExtension:@"momd"];
    _managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    return _managedObjectModel;
}
```

2 . 创建“NSPersistentStoreCoordinator”对象。

```objc
- (NSPersistentStoreCoordinator *)persistentStoreCoordinator {
    if (_persistentStoreCoordinator != nil) {
        return _persistentStoreCoordinator;
    }

	//指定需要持久化的模型对象
    _persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:[self managedObjectModel]];
    //持久化的存储文件
    NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"CoreData01.sqlite"];
    NSError *error = nil;
    //设置存储格式为SQLite
    if (![_persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error]) {

    }
    
    return _persistentStoreCoordinator;
}
```

3 . 创建上下文

```objc
- (NSManagedObjectContext *)managedObjectContext {
    // Returns the managed object context for the application (which is already bound to the persistent store coordinator for the application.)
    if (_managedObjectContext != nil) {
        return _managedObjectContext;
    }
    
    NSPersistentStoreCoordinator *coordinator = [self persistentStoreCoordinator];
    if (!coordinator) {
        return nil;
    }
    
    //创建管理上下文
    _managedObjectContext = [[NSManagedObjectContext alloc] init];
    //关联上下文与存储对象
    [_managedObjectContext setPersistentStoreCoordinator:coordinator];
    return _managedObjectContext;
}
```

4 . 设置模型文件，添加实体（Entity）

点击“CoreData01.xcdatamodelId”文件，然后添加一个实体“Book”，并增加几个属性。Core Data中的实体类似于数据库的表定义，规定了不同字段（属性）的名字和类型。

![](/images/CoreData01.png)

5 . 创建模型对象的类, "Editor > Create NSManagedobject Subclass"。

![](/images/Create_Managed_Class.png)

6 . 选择使用标量定义数值类型的属性（默认使用NSNumber类型定义int、float等类型的属性）。

![](/images/Scalar_Primary_type.png)

7 . Xcode自动创建于实体同名的类，并且继承自“NSManagedObject”。

8 . 创建对象并存储。

```objc
//获取AppDelegate中创建的上下文对象
AppDelegate *appDelegate = (AppDelegate *)[UIApplication sharedApplication].delegate;

NSManagedObjectContext *context = appDelegate.managedObjectContext;

//获取实体描述符
NSEntityDescription *entity = [NSEntityDescription entityForName:@"Book" inManagedObjectContext:context];
//创建对象
Book *book = [[NSManagedObject alloc] initWithEntity:entity insertIntoManagedObjectContext:context];
//设置对象的属性
book.title = @"红楼梦";
//保存数据
[context save:nil];
```

9 . 以后可以通过"NSFetchRequest"从文件中获取数据。

```objc
AppDelegate *appDelegate = (AppDelegate *)[UIApplication sharedApplication].delegate;
NSManagedObjectContext *context = appDelegate.managedObjectContext;

//创建请求对象，用于获取实体Book所对应的全部数据，可以通过给NSFetchRequest设置predicate和sortDescriptors对结果进行筛选和排序。
NSFetchRequest *fetchRequest = [NSFetchRequest fetchRequestWithEntityName:@"Book"];
NSArray *result = [context executeFetchRequest:fetchRequest error:nil];
NSLog(@"%@", result);
```

####总结
Core Data的简单使用还是很方便的，我们只需要关注数据内容和处理逻辑，而不需要考虑过多的存储操作。但是它需要使用很多貌似没有直接关联的代码，使得大家感觉非常复杂。

> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。

