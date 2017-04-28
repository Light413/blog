`AVCaptureDevice:`硬件设备，可以是摄像头、麦克风等。

	AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];

`AVCaptureDeviceInput`设备输入

`AVCaptureStillImageOutput`	静态图片输出

`AVCaptureSession：`	

	//session的预先设置为了适应不同质量的输出
	@property(nonatomic, copy) NSString *sessionPreset; 
	
	AVCaptureSessionPresetHigh ，default
	AVCaptureSessionPresetPhoto
	AVCaptureSessionPresetMedium
	AVCaptureSessionPresetLow
	AVCaptureSessionPreset352x288
	AVCaptureSessionPreset640x480
	AVCaptureSessionPreset1280x720
	AVCaptureSessionPreset1920x1080
	AVCaptureSessionPreset3840x2160
	AVCaptureSessionPresetiFrame960x540
	AVCaptureSessionPresetiFrame1280x720
	
`AVCaptureVideoPreviewLayer：`

	//设置在指定区域内video如何显示
	@property (copy) NSString *videoGravity;

	AVLayerVideoGravityResize				随便拉伸填充区域为止，内容会变形
	AVLayerVideoGravityResizeAspect , default 保持大小比例，适应区域，会有空白的区域
	AVLayerVideoGravityResizeAspectFill	以原比例拉伸视频，直到两边屏幕都占满，但内容有部分就被切割了；
	
@interface UIView (UIConstraintBasedLayoutCoreMethods) 

	- (void)updateConstraintsIfNeeded NS_AVAILABLE_IOS(6_0); // Updates the constraints from the bottom up for the view hierarchy rooted at the receiver. UIWindow's implementation creates a layout engine if necessary first.
	- (void)updateConstraints NS_AVAILABLE_IOS(6_0) NS_REQUIRES_SUPER; // Override this to adjust your special constraints during a constraints update pass
	- (BOOL)needsUpdateConstraints NS_AVAILABLE_IOS(6_0);
	- (void)setNeedsUpdateConstraints NS_AVAILABLE_IOS(6_0);
	
	
UIView.h

	// Allows you to perform layout before the drawing cycle happens. -layoutIfNeeded forces layout early
	- (void)setNeedsLayout;
	- (void)layoutIfNeeded;
	
	- (void)layoutSubviews;    // override point. called by layoutIfNeeded automatically. As of iOS 6.0, when constraints-based layout is used the base implementation applies the constraints-based layout, otherwise it does nothing.



UIImageOrientation
	
	typedef NS_ENUM(NSInteger, UIImageOrientation) {
    UIImageOrientationUp,            // default orientation
    UIImageOrientationDown,          // 180 deg rotation
    UIImageOrientationLeft,          // 90 deg CCW
    UIImageOrientationRight,         // 90 deg CW
    UIImageOrientationUpMirrored,    // as above but image mirrored along other axis. horizontal flip
    UIImageOrientationDownMirrored,  // horizontal flip
    UIImageOrientationLeftMirrored,  // vertical flip
    UIImageOrientationRightMirrored, // vertical flip
};

图片的方向、图片裁剪	、旋转方向

		CGContextClip(ctx);
	    UIImage *image2=[UIImage imageNamed:@"me"];
		[image2 drawAtPoint:CGPointMake(100, 100)];
	
	UIGraphicsBeginImageContextWithOptions(newSize, NO, 0.0);
    [image drawInRect:CGRectMake(0,0,newSize.width,newSize.height)];
    UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
	
	
	
二、截图功能，实现用户想要截取图的RECT

	- ( UIImage *)getImageByCuttingImage:( UIImage *)image Rect:( CGRect )rect{
	
	// 大图 bigImage
	
	// 定义 myImageRect ，截图的区域
	
	CGRect myImageRect = rect;
	
	UIImage * bigImage= image;
	
	CGImageRef imageRef = bigImage. CGImage ;
	
	CGImageRef subImageRef = CGImageCreateWithImageInRect (imageRef, myImageRect);
	
	CGSize size;
	
	size. width = rect. size . width ;
	
	size. height = rect. size . height ;
	
	UIGraphicsBeginImageContext (size);
	
	CGContextRef context = UIGraphicsGetCurrentContext ();
	
	CGContextDrawImage (context, myImageRect, subImageRef);
	
	UIImage * smallImage = [ UIImage imageWithCGImage :subImageRef];
	
	UIGraphicsEndImageContext ();
	
	return smallImage;
	
	}
	

获取脸

    for (int i = 0; i< detectResult.count; i++) {
        UIImageView * imageView = [[UIImageView alloc] initWithFrame:CGRectMake(5+105*i, 350, 100, 100)];
        CIImage * faceImage = [image imageByCroppingToRect:[[detectResult objectAtIndex:i] bounds]];
        [imageView setImage:[UIImage imageWithCIImage:faceImage]];
        [self.view addSubview:imageView];
        
    }	
    
    
    
    
    videoZoomFactor
    videoScaleAndCropFactor