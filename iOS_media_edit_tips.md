# iOS Media editing tips

## Video
### 1. fast or slow motion

```
// Your source video asset
AVURLAsset* videoAsset = [AVURLAsset assetWithURL:[[NSBundle mainBundle] URLForResource:@"test" withExtension:@"mp4"]];

//create mutable composition
AVMutableComposition *mixComposition = [AVMutableComposition composition];

AVMutableCompositionTrack *compositionVideoTrack = [mixComposition addMutableTrackWithMediaType:AVMediaTypeVideo
                                                                               preferredTrackID:kCMPersistentTrackID_Invalid];
NSError *videoInsertError = nil;
BOOL videoInsertResult = [compositionVideoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, videoAsset.duration)
                                                        ofTrack:[[videoAsset tracksWithMediaType:AVMediaTypeVideo] objectAtIndex:0]
                                                         atTime:kCMTimeZero
                                                          error:&videoInsertError];
if (!videoInsertResult || nil != videoInsertError) {
    //handle error
    return;
}

//slow down whole video by 2.0
double videoScaleFactor = 2.0; // for example：2.0 means 2x slow motion; 0.5: 2x fast motion
CMTime videoDuration = videoAsset.duration;

[compositionVideoTrack scaleTimeRange:CMTimeRangeMake(kCMTimeZero, videoDuration)
                           toDuration:CMTimeMake(videoDuration.value*videoScaleFactor, videoDuration.timescale)];

//export
NSFileManager *manager = [NSFileManager defaultManager];
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *outputURL = paths[0];
outputURL = [outputURL stringByAppendingPathComponent:@"output.mp4"];
// Remove Existing File
[manager removeItemAtPath:outputURL error:nil];
AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:mixComposition presetName:AVAssetExportPresetLowQuality];
exportSession.outputURL = [NSURL fileURLWithPath:outputURL]; // output path;
exportSession.outputFileType = AVFileTypeQuickTimeMovie;
[exportSession exportAsynchronouslyWithCompletionHandler:^(void) {
    if (exportSession.status == AVAssetExportSessionStatusCompleted) {
        [[PHPhotoLibrary sharedPhotoLibrary] performChanges:^{
            [PHAssetChangeRequest creationRequestForAssetFromVideoAtFileURL:[NSURL fileURLWithPath:outputURL]];
        } completionHandler:^(BOOL success, NSError * _Nullable error) {
            if (success) {
                // TODO: 
            }
            else {
                // TODO:
            }
        }];
    } else {
        NSLog(@"error: %@", [exportSession error]);
    }
}];
```


# image
### Converting CMSampleBuffer to a UIImage Object

The following code shows how you can convert a CMSampleBuffer to a UIImage object. You should consider your requirements carefully before using it. Performing the conversion is a comparatively expensive operation. It is appropriate to, for example, create a still image from a frame of video data taken every second or so. You should not use this as a means to manipulate every frame of video coming from a capture device in real time.

```
// Create a UIImage from sample buffer data
- (UIImage *) imageFromSampleBuffer:(CMSampleBufferRef) sampleBuffer
{
    // Get a CMSampleBuffer's Core Video image buffer for the media data
    CVImageBufferRef imageBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);
    // Lock the base address of the pixel buffer
    CVPixelBufferLockBaseAddress(imageBuffer, 0);
 
    // Get the number of bytes per row for the pixel buffer
    void *baseAddress = CVPixelBufferGetBaseAddress(imageBuffer);
 
    // Get the number of bytes per row for the pixel buffer
    size_t bytesPerRow = CVPixelBufferGetBytesPerRow(imageBuffer);
    // Get the pixel buffer width and height
    size_t width = CVPixelBufferGetWidth(imageBuffer);
    size_t height = CVPixelBufferGetHeight(imageBuffer);
 
    // Create a device-dependent RGB color space
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
 
    // Create a bitmap graphics context with the sample buffer data
    CGContextRef context = CGBitmapContextCreate(baseAddress, width, height, 8,
      bytesPerRow, colorSpace, kCGBitmapByteOrder32Little | kCGImageAlphaPremultipliedFirst);
    // Create a Quartz image from the pixel data in the bitmap graphics context
    CGImageRef quartzImage = CGBitmapContextCreateImage(context);
    // Unlock the pixel buffer
    CVPixelBufferUnlockBaseAddress(imageBuffer,0);
 
    // Free up the context and color space
    CGContextRelease(context);
    CGColorSpaceRelease(colorSpace);
 
    // Create an image object from the Quartz image
    UIImage *image = [UIImage imageWithCGImage:quartzImage];
 
    // Release the Quartz image
    CGImageRelease(quartzImage);
 
    return (image);
}
```


