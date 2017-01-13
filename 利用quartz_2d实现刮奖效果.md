####利用Quartz 2D实现刮奖效果
刮奖是商家类项目中经常使用的组件，其实现方式也有多种，下面介绍一种使用Quartz 2D实现的一种方式。

---

Quartz 2D中有一种Masking Images(*有图片遮罩的意思*)，下面看一个官方文档给出的效果来一个直观的展示。
<img src="file:/Users/a58/Desktop/MarkDown/lotteryoriginal.png" width = 100%>

<center>原图</center>

<img src="file:/Users/a58/Desktop/MarkDown/lotterymask.png" width = 100%>

<center>遮罩图</center>

<img src="file:/Users/a58/Desktop/MarkDown/lotteryresult.png" width = 100%>

<center>结果图</center>
从中可以看到黑色的部分将原图显示了出来，白色的部分把原图遮住了，灰色的部分和原图经过一定的算法进行了合成。我们可以想象不断的改变遮罩层来改变最后的合成效果，最终产生刮奖的效果。具体步骤如下：

<img src="file:/Users/a58/Desktop/MarkDown/lotteryworkflow.png" heigth = 50% width = 70% align=center>

具体的代码如下:


    CGColorSpaceRef colorspace = CGColorSpaceCreateDeviceGray();
    float scale = [UIScreen mainScreen].scale;
    //1. 获取刮奖层
    UIGraphicsBeginImageContextWithOptions(hideView.bounds.size, NO, 0);
    [hideView.layer renderInContext:UIGraphicsGetCurrentContext()];
    hideView.layer.contentsScale = scale;
    hideImage = UIGraphicsGetImageFromCurrentImageContext().CGImage;
    UIGraphicsEndImageContext();
    
    size_t imageWidth = CGImageGetWidth(hideImage);
    size_t imageHeight = CGImageGetHeight(hideImage);
    CFMutableDataRef pixels = CFDataCreateMutable(NULL, imageWidth * imageHeight);
    //2. 手指滑动的时候不断在这个context上画上白线。
    contextMask = CGBitmapContextCreate(CFDataGetMutableBytePtr(pixels), imageWidth, imageHeight , 8, imageWidth, colorspace, kCGImageAlphaNone);
    CGContextFillRect(contextMask, self.frame);
    // 设置滑动时候产生的线条颜色是白色
    CGContextSetStrokeColorWithColor(contextMask, [UIColor whiteColor].CGColor);
    CGContextSetLineWidth(contextMask, _sizeBrush);
    CGContextSetLineCap(contextMask, kCGLineCapRound);
    
    CGDataProviderRef dataProvider = CGDataProviderCreateWithCFData(pixels);
    CGImageRef mask = CGImageMaskCreate(imageWidth, imageHeight, 8, 8, imageWidth, dataProvider, nil, NO);
    
    //2. 根据iamge mask产生最终的图片
    scratchImage = CGImageCreateWithMask(hideImage, mask);
    CGImageRelease(mask);
    CGColorSpaceRelease(colorspace);
    
手指滑动时候调用的方法:


    - (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesMoved:touches withEvent:event];

    UITouch *touch = [[event touchesForView:self] anyObject];
    currentTouchLocation = [touch locationInView:self];
    previousTouchLocation = [touch previousLocationInView:self];
    [self scratchTheViewFrom:previousTouchLocation to:currentTouchLocation];
    //获取touch对象
    UITouch *t = touches.anyObject;
    [self recordPassRect:t];
    
    }
    
    // 绘制图像
    - (void)scratchTheViewFrom:(CGPoint)startPoint to:(CGPoint)endPoint {
    
    BOOL needRender = [self needRenderWithCurrentLocation:endPoint previousLocation:previousTouchLocation];
    if (!needRender) return;
    
    float scale = [UIScreen mainScreen].scale;
    CGContextMoveToPoint(contextMask, startPoint.x * scale, (self.frame.size.height - startPoint.y) * scale);
    CGContextAddLineToPoint(contextMask, endPoint.x * scale, (self.frame.size.height - endPoint.y) * scale);
    CGContextStrokePath(contextMask);
    // 调用drawRect 方法
    [self setNeedsDisplay];
    self.isDrawn = YES;
    
    }
    
    - (void)drawRect:(CGRect)rect {
    
    UIImage *imageToDraw = [UIImage imageWithCGImage:scratchImage];
    [imageToDraw drawInRect:CGRectMake(0.0, 0.0, self.frame.size.width, self.frame.size.height)];
    }

    
    

