# iOS multi threads tips

### 1. GCD timer
```
- (void)gcdTimer {
 
    // get the queue
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
     
    // creat timer
    self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    // config the timer (starting time，interval)
    // set begining time
    dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
    // set the interval
    uint64_t interver = (uint64_t)(1.0 * NSEC_PER_SEC);
     
    dispatch_source_set_timer(self.timer, start, interver, 0.0);
     
    dispatch_source_set_event_handler(self.timer, ^{
        // the tarsk needed to be processed async
        dispatch_async(dispatch_get_global_queue(0, 0), ^{
            NSLog(@"gcdTimer");
        });
    });
     
    dispatch_resume(self.timer);
}
```


### 2.其他线程中 performSelector 无效
栗子：

```    
- (void)test {
    dispatch_async(dispatch_get_global_queue(0, DISPATCH_QUEUE_PRIORITY_DEFAULT), ^{
        [self performSelector:@selector(test1) withObject:nil afterDelay:1];
        
        // 解决方法1，开启子线程的runloop
        //[[NSRunLoop currentRunLoop] run];
        
        // 解决方法2，回到主线程 performSelector
        //dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)1*NSEC_PER_SEC), dispatch_get_main_queue(), ^{
                //[self performSelector:@selector(test1) withObject:nil afterDelay:1];
            //});
        //});
}

- (void)test1 {
    NSLog(@"------->>>>>>>>>>>> test1");
}
```

