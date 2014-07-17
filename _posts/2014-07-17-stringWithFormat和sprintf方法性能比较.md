---
layout: post
##### title: StringWithFormat和sprintf方法性能比较
---

TableView滚动有点卡的时候，在保证Core Animation帧数是60的情况下，有时还要再看一下Time Profile。比如哪个是当前比较耗时的方法，分析一下原因以及解决方法。


昨天解决Detail页面滚动评论卡的时候，Core Animation的帧数始终在59-60之间，貌似看不出什么问题。
![](/AllenChiangBlog/public/upload/2014-07-17-coreanimation.png)


再看一下Time Profile貌似发现有一个比较常见的[NSString stringWithFormat:]方法居然占到了13.7%的执行时间感觉有点不靠谱。
![](/AllenChiangBlog/public/upload/2014-07-17-timeprofile.png)


### 定位原因

Detail页面的滚动是用户评论，还好内容不多；用排除法最终定位在评论内容做表情替换这里，代码如下：
![](/AllenChiangBlog/public/upload/2014-07-17-replaceemoj.png)

解释一下，因为表情有100多个，而且服务端返回的是类似/:026、/:-W之类的伪符号，所以循环把这些伪符号替换成一段类似html的代码，并且每次循环都用到[NSString stringWithFormat:]方法构建了这段类似html的代码。

### 解决方案

1. 可以把上述字符串替换的过程放到global线程中执行，等替换成功了再切换回来执行真正的绘图相关代码。

2. 既然[NSString stringWithFormat:]这么坑，我们就想是否有性能更好的方法做同样的事情。
	int sprintf( char *restrict buffer, const char *restrict format, ... );    (since C99)

所以上面这段表情替换的代码就可以改成

	- (void)setText:(NSString *)text {
		NSString *s = [text copy];
   		NSDictionary *dict = [FMCommon getEmojiToImageDict];
   		char emojiKey[250];
   		CGFloat pointSize = self.font.pointSize ;
		for (NSString *key in [dict allKeys]) {
        		NSString *value = [dict objectForKey:key];
        		if (value) {
            		sprintf(emojiKey, EMOJI_DIRECTORY, [value UTF8String],pointSize,pointSize);
            		NSString *imageValue = [NSString stringWithCString:emojiKey encoding:NSASCIIStringEncoding];
            		s = [s stringByReplacingOccurrencesOfString:key
                                             withString:imageValue];
        		}
    		}
    		[super setText:s];
	}

现在我们Detail页面是上述两种方法一起应用，改成这样以后肉眼看滚动TableView似乎没那么卡了，再看一下TimeProfile终于也没有占用特别长时间的方法了。

### 性能对比

用[NSString stringWithFormat:]和sprint()两种方法先后进行一次总计1million的字符串拼装，分别看看两者的耗时情况
	long calCount = 1000000;

#### stringWithFormat
	for (int i=0; i<calCount; i++) {
                testString = [NSString stringWithFormat:AlTStringTestStringFormat,sample,i];
            }

#### sprintf()
	char testchar[20];
      const char* sampleChars = [sample UTF8String];
      for (int i=0; i<calCount; i++) {
             sprintf(testchar,sampleChars ,i);
       }

同样循环1百万次拼装字符串的性能差距到底有大，我们通过计算程序执行的时间来看

	   mach_timebase_info_data_t info;
        if (mach_timebase_info(&info) != KERN_SUCCESS) return -1.0;
        
        uint64_t start = mach_absolute_time ();
        // 这里写入需要统计时间的执行代码
        uint64_t end = mach_absolute_time ();
        uint64_t elapsed = end - start;
        uint64_t nanos = elapsed * info.numer / info.denom;
        return (CGFloat)nanos / NSEC_PER_SEC;

#### 对比结果
在iPhone 5真机上面执行上述程序的结果：

- stringWithFormat 执行这1million的时间是7.025311秒

- sprintf执行同样这1million的时间是0.326543秒

但从对比结果上来看sprintf的效率应该是stringWithFormat的20倍以上，看来以后对于大量的字符串拼装最好还是用sprintf方法。

至于为什么是这样，因为还没有找到stringWithFormat方法的真正源码，还为从得知，有知晓的还望指教。