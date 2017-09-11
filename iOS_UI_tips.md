# iOS UI tips
### 1. 扩大按钮响应范围

```
//First, add a category to UIButton that overrides the hit test and also adds a property for expanding the hit test frame.

//UIButton+Extensions.h
@interface UIButton (Extensions)
@property(nonatomic, assign) UIEdgeInsets hitTestEdgeInsets;
@end

//UIButton+Extensions.m
#import "UIButton+Extensions.h"
#import <objc/runtime.h>

@implementation UIButton (Extensions)

@dynamic hitTestEdgeInsets;

static const NSString *KEY_HIT_TEST_EDGE_INSETS = @"HitTestEdgeInsets";

-(void)setHitTestEdgeInsets:(UIEdgeInsets)hitTestEdgeInsets {
    NSValue *value = [NSValue value:&hitTestEdgeInsets withObjCType:@encode(UIEdgeInsets)];
    objc_setAssociatedObject(self, &KEY_HIT_TEST_EDGE_INSETS, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

-(UIEdgeInsets)hitTestEdgeInsets {
    NSValue *value = objc_getAssociatedObject(self, &KEY_HIT_TEST_EDGE_INSETS);
    if(value) {
        UIEdgeInsets edgeInsets; [value getValue:&edgeInsets]; return edgeInsets;
    }else {
        return UIEdgeInsetsZero;
    }
}

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    if(UIEdgeInsetsEqualToEdgeInsets(self.hitTestEdgeInsets, UIEdgeInsetsZero) ||       !self.enabled || self.hidden) {
        return [super pointInside:point withEvent:event];
    }

    CGRect relativeFrame = self.bounds;
    CGRect hitFrame = UIEdgeInsetsInsetRect(relativeFrame, self.hitTestEdgeInsets);

    return CGRectContainsPoint(hitFrame, point);
}

@end

//Once this class is added, all you need to do is set the edge insets of your button. 
//Note that I chose to add the insets so if you want to make the hit area larger, you must use negative numbers.

// usage
[button setHitTestEdgeInsets:UIEdgeInsetsMake(-10, -10, -10, -10)];
//Note: Remember to import the category (#import "UIButton+Extensions.h") in your classes.
```

