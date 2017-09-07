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

