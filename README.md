# ZBNetworking    [介绍文档](http://www.jianshu.com/p/55cda3341d11)
 
 本来想发布到cocoaPods的发现名字已经被占用了，也不想改名了。大家就手动下载用吧

[变更日志](https://github.com/Suzhibin/ZBNetworking/blob/master/CHANGELOG)
 
优点:

1.请求类型丰富 /**GET请求*//**POST请求*//**PUT请求*//**PATCH请求*//**DELETE请求*//**Upload请求*//**DownLoad请求*/

2.低耦合，易扩展。

3.通过Block配置信息，代码紧凑;

4.有缓存文件过期机制 默认一周

5.显示缓存大小/个数，全部清除缓存/单个文件清除缓存/按时间清除缓存/按路径清除缓存  方法多样  并且都可以自定义路径   可扩展性强

6.有缓存key过滤功能

7.离线下载功能 

8.多种请求缓存类型的判断。也可不遵循，自由随你定。

```objective-c
   /** 重新请求 ,不读取缓存，重新请求*/
    ZBRequestTypeRefresh,
    /** 有缓存,读取缓存 无缓存，重新请求*/
    ZBRequestTypeCache,
    /** 加载更多 ,不读取缓存，重新请求*/
    ZBRequestTypeRefreshMore,
    /** 加载更多 ,有缓存,读取缓存 无缓存，重新请求*/
    ZBRequestTypeCacheMore,
    /** 详情    ,有缓存,读取缓存 无缓存，重新请求*/
    ZBRequestTypeDetailCache,
    /** 自定义  ,有缓存,读取缓存 无缓存，重新请求*/
    ZBRequestTypeCustomCache
```
9.可见的缓存文件

![](http://a3.qpic.cn/psb?/V12I5WUv0Ual5v/uls*nG1YySR.EpyYI8*lFu9kW.lwzjgW.cnPbGMUBG8!/b/dPgAAAAAAAAA&bo=aAHwAAAAAAACDLE!&rf=viewer_4)

## 使用 
```objective-c
//请求方法 会默认创建缓存路径    
  [ZBRequestManager requestWithConfig:^(ZBURLRequest *request){
        request.URLString=list_URL;
        request.methodType=ZBMethodTypeGET;//默认为GET
        request.apiType=ZBRequestTypeRefresh;//默认为刷新  //ZBRequestTypeCache为使用缓存
        request.timeoutInterval=10;//默认为30
       // request.requestSerializer = ZBJSONRequestSerializer;根据自己服务器的要求调整上传参数的格式
       // request.responseSerializer= ZBJSONResponseSerializer;得到的返回数据 格式  默认为json  
       // request.parameters=@{@"1": @"one", @"2": @"two"};
       // [request setValue:@"1234567890" forHeaderField:@"apitype"];
    }  success:^(id responseObj,apiType type){
        if (type==ZBRequestTypeRefresh) 
             //结束刷新
        }
        if (type==ZBRequestTypeLoadMore) {
            //结束上拉加载
        }
        //请求成功
          NSLog(@"得到数据:%@",responseObject);
      
    } failure:^(NSError *error){
        if (error.code==NSURLErrorCancelled)return;
        if (error.code==NSURLErrorTimedOut){
            [self alertTitle:@"请求超时" andMessage:@""];
        }else{
            [self alertTitle:@"请求失败" andMessage:@""];
        }
    }];

```

## 使用 其他功能
1.离线下载 批量下载


```objective-c
 [ZBRequestManager sendBatchRequest:^(ZBBatchRequest *batchRequest)
            for (NSString *urlString in offlineArray) {
            ZBURLRequest *request=[[ZBURLRequest alloc]init];
            request.URLString=urlString;
            [batchRequest.urlArray addObject:request];
        }
    }  success:^(id responseObj,apiType type){
      
    } failure:^(NSError *error){
        if (error.code==NSURLErrorCancelled)return;
        if (error.code==NSURLErrorTimedOut){
            [self alertTitle:@"请求超时" andMessage:@""];
        }else{
            [self alertTitle:@"请求失败" andMessage:@""];
        }
    }];

//具体演示看demo
```
![](http://a3.qpic.cn/psb?/V12I5WUv0Ual5v/cY8K3L2*GJ9RO3i*z1If9XTmzas0cylmafMXWqdFe4o!/b/dK0AAAAAAAAA&bo=aAHwAAAAAAACLJE!&rf=viewer_4)


2.缓存相关
```objective-c
 //显示缓存大小 可以自定义路径
 [[ZBCacheManager sharedInstance]getCacheSize];
  //显示缓存个数  可以自定义路径
 [[ZBCacheManager sharedInstance]getCacheCount];
  //清除缓存
 [[ZBCacheManager sharedInstance]clearCache];
  //清除单个缓存文件
 [[ZBCacheManager sharedInstance]clearCacheForkey:list_URL];
  //按路径清除缓存
 [[ZBCacheManager sharedInstance]clearDiskWithpath:@"路径" completion:nil];
  //取消当前请求
 [ZBRequestManager cancelRequest:_urlString completion:^(NSString *urlString){
      //NSLog(@"取消对应url:%@ ",urlString);
  }];
  
 ```

![](https://upload-images.jianshu.io/upload_images/1830250-3636c0621ebb6fa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/621)

## 缓存key过滤
 ```
    NSDate* dat = [NSDate dateWithTimeIntervalSinceNow:0];
    NSTimeInterval a=[dat timeIntervalSince1970];
    NSString *timeString = [NSString stringWithFormat:@"&time=%f", a];

    //作者遇到到请求 是在get请求后加一个时间戳的参数，因为URLString 是默认为缓存key的 加上时间戳，key 一直变动 无法拿到缓存。所以定义了一个customCacheKey
      NSTimeInterval timeInterval = [[NSDate date] timeIntervalSince1970];
    NSString *timeString = [NSString stringWithFormat:@"&time=%f", timeInterval];

    [ZBRequestManager requestWithConfig:^(ZBURLRequest *request){
        request.URLString=[list_URL stringByAppendingString:timeString];
        request.customCacheKey=list_URL;//去掉timeString
        request.methodType=ZBMethodTypeGET;
        request.apiType=ZBRequestTypeCache;//默认为ZBRequestTypeRefresh
    }  success:^(id responseObject, apiType type, BOOL isCache) {
        if (isCache) {
            NSLog(@"使用了缓存");
        }else{
            NSLog(@"重新请求");
        }
    
    }  failure:nil];
    
    
     //POST等 使用了parameters 的请求 缓存key会是URLString+parameters，parameters里有是时间戳或者其他动态参数,key一直变动 无法拿到缓存。所以定义一个parametersfiltrationCacheKey 过滤掉parameters 缓存key里的 变动参数比如 时间戳
    [ZBRequestManager requestWithConfig:^(ZBURLRequest *request){
        request.URLString=@"http://URL";
        request.methodType=ZBMethodTypePOST;//默认为GET
        request.apiType=ZBRequestTypeRefresh;//默认为ZBRequestTypeRefresh
        request.parameters=@{@"1": @"one", @"2": @"two", @"time": @"12345667"};
        request.parametersfiltrationCacheKey=@[@"time"];//过滤掉parameters 缓存key里
    }success:nil failure:nil];
  ```


## License

ZBNetworking is released under the MIT license. See LICENSE for details.
