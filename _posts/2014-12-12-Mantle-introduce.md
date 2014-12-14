---
layout: post
category : 翻译
tagline: "Supporting tagline"
tags : [Mantle, 模型, JSON, Objective-C]
---
#Mantle
**Mantle**是一个iOS模型框架，它为对象和JSON之间的相互转化提供了一种简便的方法。这在处理网络数据的时候非常有用。
下面我们将看一下`MTLModel`、`MTLJSONAdapter`以及为什么你将会考虑在下一个项目中使用Mantle。

##MTLModel
`MTLModel`提供一个简便的方法在`NSDictionary`对象和自定义对象之间建立映射关系。首先我们来看一个例子。假设从远程服务器上获取了如下JSON数据，然后我们需要将它转化为自定义类型`CATProfile`的对象。

```js
{
	"id": 1,
    "name":"objective Cat",
    "birthday":"2013-09-12 13:29:36 +0100",
    "website":"http://objc.at",
    "location":{"lat":"48.2083","lon":"16.3731"},
    "relationship_status":"single",
    "awesome":true
}
```

接下来创建一个`MTLModel`的子类用来表示上面的JSON对象。

```objc
//CATProfile.h
typedef NS_ENUM(NSInteger, CATRelationshipStatus) {
	CATRelationshipStatusSingle = 0,
    CATRelationshipStatusRelationship,
    CATRelationshipStatusComplicated
};

@interface CATProfile : MTLModel <MTLJSONSerializing>
@property (strong, nonatomic) NSNumber *profileId;
@property (strong, nonatomic) NSString *name;
@property (strong, nonatomic) NSDate *birthday;
@property (strong, nonatomic) NSURL *websiteURL;
@property (nonatomic) CLLocationCoordinate2D locationCoordinate;
@property (nonatomic) CATRelationshipStatus relationshipStatus;
@property (nonatomic, getter=isAwesome) BOOL awesome;
@end
```

`CATProfile`类继承自`MTLModel`并且实现了`MTLJSONSerializing`协议。该协议要求实现`+JSONKeyPathsByPropertykey`方法。

```objc
//CATProfile
@implementation CATProfile

+ (NSDictionary *)JSONKeyPathsByPropertyKey {
	//头文件中定义的属性  < : > JSON字典中的key
	return @{
    	@"profileId": 	@"id",
        @"websiteURL": 	@"website",
        @"locationCoordinate": @"location",
        @"relationshipStatus": @"relationship_status",
    };
}

@end
```

`+JSONKeyPathsByPropertyKey`返回一个`NSDictionary`用来建立JSON和模型对象属性之间的映射。如果模型属性没有出现在上面的字典里，Mantle自动将它与JSON中同名的值对应起来。

##NSValueTransformer
Mantle可以自动处理`NSString`和`NSNumber`类型的数据，而对其它数据类型的值则需要我们提供一定的帮助才能正确转换。Mantle利用Foundation框架中的`NSValueTransformer`在JSON数据和实际模型属性之间建立映射。
我们通过实现名为`+<propertyName>JSONTransformer`的类方法返回所需要的`NSValueTransformer`对象。

```objc
//将birthday映射到NSDate，反之亦然
+ (NSValueTransformer *)birthdayJSONTransformer {
	return [MTLValueTransformer reversibleTransfomerWithForwardBlock: ^(NSString *dateString) {
    	return [self.dateFormatter dateFromString: dateString];
    } reverseBlock:^(NSDate *date) {
    	return [self.dateFormatter stringFromDate: date];
    }];
}

+ (NSDateFormatter *)dateFormatter {
	NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    dateFormatter.locale = [[NSLocale alloc] initWithLocaleIndentifier: @"en_US_POSIX"];
    dateFormatter.dateFormat = @"yyyy-MM-dd 'T' HH:mm:ss.SSS 'Z'";
    
    return dateFormatter;
}
```

Mantle在运行时调用该方法来决定如何转换birthday属性。下面是其它类型属性的转换器方法。
**NSURL <->JSON string**

```objc
+ (NSValueTransformer *)websiteURLJSONTransformer {
	return [NSValueTransformer valueTransformerForName: MTLURLValueTransformerName];
}
```

**CLLocationCoordinate2D <-> JSON object**

```objc
+ (NSValueTransformer *)locationCoordinateJSONTransformer {
	return [MTLValueTranformer reversibleTransformerWithForwardBlock: ^(NSDictionary *coordicateDict){
    CLLocationDegree latitude = [coordinateDict[@"lat"] doubleValue];
    CLLocationDegree longitude = [coordinateDict[@"lon"] doubleValue];
    
    return [NSValue valueWithMKCoordinate: CLLocationCoordinate2DMake(latitude, longitude)];
    } reverseBlock:^(NSValue *coordinateValue){
    	CLLocationCoordinate2D coordinate = [coordinateValue MKCoordinateValue];
        return @{@"lat":@(coordinate.latitude), @"lon":@(coordinate.longitude)};
    }];
}
```

**enum <-> JSON string**

```objc
+ (NSValueTransformer *)relationshipStatusJSONTransformer {
	return [NSValueTransformer mtl_valueMappingTransformerWithDictionary: @{
    	@"single":@(CATRelationshipStatusSingle),
        @"relationship":@(CATRelationshipStatusInRelationship),
        @"complicated":@(CATRelationshipStatusComplicated)
    }];
}
```

**BOOL <-> JSON boolean**

```objc
+ (NSValueTransformer *)awesomeJSONTransformer {
	return [NSValueTransformer valueTransformerForName: MTLBooleanValueTransformerName];
}
```

##从JSON创建模型对象
我们已经配置好了模型属性与JSON值的映射关系，可以开始将从API获取到的数据转换为模型对象了。第一步需要用`NSJSONSerialization`将JSON表示解析为`NSDictionary`供Mantle使用。然后通过`MTLJSONAdapter`将字典转换为模型对象。

```objc
//创建NSDictionary
NSData *JSONData = ...//接口的响应数据
NSDictionary *JSONDict = [NSJSONSerialization JSONObjectWithData: JSONData options: 0 error: nil];

//使用MTLJSONSerialization创建模型对象
CATProfile *profile = [MTLJSONAdapter modelOfClass: CATProfile.class fromJSONDictionary: JSONDict error: NULL];
```

##从模型对象创建JSON
`MTLJSONAdapter`同样可以用来从模型对象创建字典。

```objc
CATProfile *profile = ...
NSDictionary *profileDict = [MTLJSONAdapter JSONDictionaryFromModel: profile];

NSData *JSONData = [NSJSONSerialization dataWithJSONObject: profileDict options: 0 error: nil];
```

> 注意，如果模型中的属性如果没有对应的JSON值，应该在`+JSONKeyPathsByPropertyKey`的返回值中指定为`NSNull.null`，例如`@{"name":NSNull.null}`，这样Mantle会自动忽略它。

##映射数组和字典
许多模型相互之间都有关联，它们一般通过JSON数组和对象来表示。

```js
{
	"id": 1,
    "name": "Objective Cat",
    ...
    
    "owner": {
    	"id":99,
        "name":"Alexander Schuch"
    },
    "friends":[
    	{
        	"name":"Owly",
            "type":"bird"
        },
        {
        	"name":"Hedgy",
            "type":"mammal"
        }
    ]
}
```

Mantle同样能很好的完成这些对象之间的映射。为了确保正确映射， 我们可以使用`NSValueTransformer`类别中的两个方法。

```objc
+ (NSValueTransformer *)mtl_JSONDictionaryTransformerWithModelClass:(Class)modelClass;
+ (NSValueTransformer *)mtl_JSONArrayTransformerWithModelClass:(Class)modelClass;
```

当然，同样需要在模型类`CATProfile`中创建转换方法。

```objc
//CATProfile.h
@property (strong, nonatomic) CATOwner *owner;
@property (strong, nonatomic) NSArray *friends;

//CATProfile.m
+ (NSValueTransformer *)ownerJSONTransformer {
	return [NSValueTransfomer mtl_JSONDictionaryTransfomerWithModelClass: CATOwner.class];
}

+ (NSValueTransformer *)friendsJSONTransfrmer {
	return [NSValueTransformer mtl_JSONArrayTransformerWithMocelClass: CATFriend.class];
}
```

来源：[http://www.objc.at/mantle](http://www.objc.at/mantle)
