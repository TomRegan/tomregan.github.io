+++
date = '2012-12-09'
title = 'Lazy Instantiation in Objective C'
+++

Lazy instantiation is a relatively cheap way of allocating memory to
component objects as it's required instead of up-front in a constructor.

{% highlight objectivec %}
@interface MyStack()
@property (nonatomic, strong) NSMutableArray *stack;
@end
 
@implementation MyStack
 
@synthesize stack = _stack;
 
- (NSMutableArray*)stack
{
    if (_stack == nil) {
        _stack = [[NSMutableArray alloc] init];
    }
    return _stack;
}
- (void)push:(double)operand
{
    [self.stack addObject: [NSNumber numberWithDouble:operand]];
}
@end
{% endhighlight %}

This implementation overrides the synthesized accessor and adds an
alloc-init call if the storage hasn't yet been initialized.

Initialization happens at line 12, when the first message is sent to
`addObject:` by `push:`, which means that no storage is allocated until it's required
by the application.
