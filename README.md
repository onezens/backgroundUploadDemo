# YCUploadSession


### Background upload 
1. background stream upload from file

    ```
    - (void)bgUploadStreamFile {
        
        NSLog(@"%s", __func__);
        NSURL *url = [NSURL URLWithString:kStreamUploadUrl];
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        [request setHTTPMethod:@"POST"];
        NSString *path = [[NSBundle mainBundle]pathForResource:@"icon" ofType:@"jpg"];
        NSFileManager *fileMgr = [NSFileManager defaultManager];
        NSDictionary *fileAttri = [fileMgr attributesOfItemAtPath:path error:nil];
        NSNumber *fileSize = [fileAttri valueForKey:@"NSFileSize"];
        request.HTTPBodyStream = [NSInputStream inputStreamWithFileAtPath:path];
        [request setValue:[NSString stringWithFormat:@"%zd", fileSize.integerValue] forHTTPHeaderField:@"Content-Length"];
        [request setValue:@"application/octet-stream" forHTTPHeaderField:@"Content-Type"];
        self.uploadTask = [self.backgroundSession uploadTaskWithStreamedRequest:request];
        [self.uploadTask resume];
    }
    ```
2. background stream upload with form data

    ```
    - (void)bgUploadStreamForm
    {
        NSLog(@"%s", __func__);
        NSURL *url = [NSURL URLWithString:kFormUploadUrl];
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        [request setHTTPMethod:@"POST"];
        NSString *path = [[NSBundle mainBundle]pathForResource:@"icon" ofType:@"jpg"];
        NSData *bodydata = [self buildBodyDataWithPicPath:path];
        NSString *contentType = [NSString stringWithFormat:@"multipart/form-data; charset=utf-8;boundary=%@",boundary];
        [request setValue:contentType forHTTPHeaderField:@"Content-Type"];
        [request setValue:[NSString stringWithFormat:@"%zd", bodydata.length] forHTTPHeaderField:@"Content-Length"];
        request.HTTPBodyStream = [NSInputStream inputStreamWithData:bodydata];
        self.uploadTask = [self.backgroundSession uploadTaskWithStreamedRequest:request];
        [self.uploadTask resume];
    }
    ```
3. background upload from file

    ```
    - (void)bgUploadFromFile{
        NSLog(@"%s", __func__);
        NSURL *url = [NSURL URLWithString:kStreamUploadUrl];
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        [request setHTTPMethod:@"POST"];
        NSString *path = [[NSBundle mainBundle] pathForResource:@"icon.jpg" ofType:nil];
        self.uploadTask = [self.backgroundSession uploadTaskWithRequest:request fromFile:[NSURL fileURLWithPath:path]];
        [self.uploadTask resume];
    }
    ```

### Normal upload


```
-(void)upload {
 NSLog(@"%s", __func__);
    NSURL *url = [NSURL URLWithString:kFormUploadUrl];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    [request setHTTPMethod:@"POST"];
    NSString *contentType = [NSString stringWithFormat:@"multipart/form-data; charset=utf-8;boundary=%@",boundary];
    [request setValue:contentType forHTTPHeaderField:@"Content-Type"];
    NSString *path = [[NSBundle mainBundle]pathForResource:@"icon" ofType:@"jpg"];
    NSData *bodydata = [self buildBodyDataWithPicPath:path];
    NSURLSession *session = [NSURLSession sharedSession];
    NSURLSessionUploadTask *task = [session uploadTaskWithRequest:request fromData:bodydata completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        //打印出响应体，查看是否发送成功
        NSLog(@"response = %@",response);
        
    }];
    [task resume];
  
}


-(NSData*)buildBodyDataWithPicPath:(NSString *)path{
    
    NSMutableData *bodyData = [NSMutableData data];
    NSMutableString *bodyStr = [NSMutableString string];
    [bodyStr appendFormat:@"--%@\r\n",boundary];//\n:换行 \n:切换到行首
    [bodyStr appendFormat:@"Content-Disposition: form-data; name=\"sampleFile\"; filename=\"icon.jpg\""];
    [bodyStr appendFormat:@"\r\n\r\n"];
    
    NSData *start = [bodyStr dataUsingEncoding:NSUTF8StringEncoding];
    [bodyData appendData:start];
    NSData *picData = [NSData dataWithContentsOfFile:path];
    [bodyData appendData:picData];

    bodyStr = [NSMutableString string];
    [bodyStr appendFormat:@"\r\n--%@--",boundary];
    
    NSData *endData = [bodyStr dataUsingEncoding:NSUTF8StringEncoding];
    [bodyData appendData:endData];
    return bodyData;
    
}
```

