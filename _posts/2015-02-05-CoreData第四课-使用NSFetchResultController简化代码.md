---
layout: post
category : 原创
tagline: "Supporting tagline"
tags : [iOS, Core Data, 数据存储, Mac, 应用开发]
---
####问题
如果Core Data存储的数据发生改变了，怎样才能通知界面做出响应？如果数据需要分组，如何处理？

####解决方法
Core Data中提供了`NSFetchResultController`，可以帮助我们简化数据层与UI层的交互，尤其是与`UITableView`之间。`NSFetchResultController`帮我们将数据用类似`UITableView`的dataSource所使用的数据结构，由section和row组成。并且它能够通过代理方法将数据的修改事件发布出来，让我们可以及时的更新UI。

####使用步骤

1 .创建`NSFetchResultController`对象。它只是帮我们管理所获取到的数据结果。因此在创建`fetchResultController`对象时，需要先创建`NSFetchRequest`对象，并且必须拥有`sortDescriptor`。

```objc
AppDelegate *appDelegate = [AppDelegate appDelegate];

NSFetchRequest *fetchRequest = [NSFetchRequest fetchRequestWithEntityName:@"Student"];

//对结果进行排序
NSSortDescriptor *desc = [NSSortDescriptor sortDescriptorWithKey:@"name" ascending:YES];
fetchRequest.sortDescriptors = @[desc];

//创建Controller对象，需要设置按哪个属性进行分组（section），属性值相同的归为同一组
resultCtrl = [[NSFetchedResultsController alloc] initWithFetchRequest:fetchRequest managedObjectContext:appDelegate.managedObjectContext sectionNameKeyPath:@"gender" cacheName:nil];

//设置代理对象
resultCtrl.delegate = self;
```

2 .实现常用的代理方法，获取数据改变或section改变的事件。

```objc
- (void)controller:(NSFetchedResultsController *)controller didChangeObject:(id)anObject atIndexPath:(NSIndexPath *)indexPath forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(NSIndexPath *)newIndexPath {
	//anObject为发生了改变的对象，比如修改、插入、删除等
    NSLog(@"%@", anObject);
}
```

3 .获取数据。

```objc
//开始获取数据，如果成功，返回YES
[resultCtrl performFetch:nil];
//fetchedObjects是所有符合fetchRequest的结果
NSArray *arr = resultCtrl.fetchedObjects;
for (Student *stu in arr) {
    NSLog(@"%@:%d:%d", stu.name, stu.age, stu.gender);
}

NSLog(@"------");
//按组分，每个分组的对象实现了`NSFetchedResultsSectionInfo`协议
//可以得到每组数据的对象：obj.objects, 个数：obj.numberOfObjects，值：obj.name
for (id<NSFetchedResultsSectionInfo> obj in resultCtrl.sections) {
    NSLog(@"%@", obj.indexTitle);
}
```

4 .每次发生改变都会调用代理方法。

```objc
NSEntityDescription *entity = [NSEntityDescription entityForName:@"Student" inManagedObjectContext:appDelegate.managedObjectContext];
//创建新的Student对象
Student *student = [[Student alloc] initWithEntity:entity insertIntoManagedObjectContext:appDelegate.managedObjectContext];
student.name = @"Zhangsan";

uint32_t gender = arc4random_uniform(100) % 2;
student.gender = gender?YES:NO;

student.age = arc4random_uniform(200);

//插入数据
[appDelegate.managedObjectContext insertObject:student];
//记得保存，否则不会真正写入文件
[appDelegate.managedObjectContext save:nil];
```

5 .在使用`NSFetchedResultsController`时，由于对数据进行分组需要消耗较多的资源，所以可以将查询结果缓存起来。我们在创建`NSFetchedResultsController`对象的时候，传递一个cacheName就可以了。

```objc
resultCtrl = [[NSFetchedResultsController alloc] initWithFetchRequest:fetchRequest managedObjectContext:appDelegate.managedObjectContext sectionNameKeyPath:@"gender" cacheName:@"studentGender"];
```

> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。

