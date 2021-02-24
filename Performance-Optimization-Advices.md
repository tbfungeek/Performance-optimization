2.合理的线程分配
由于 GCD 实在太方便了，如果不加控制，大部分需要抛到子线程操作都会被直接加到 global 队列，这样会导致两个问题，1.开的子线程越来越多，线程的开销逐渐明显，因为开启线程需要占用一定的内存空间（默认的情况下，主线程占1M,子线程占用512KB）。2.多线程情况下，网络回调的时序问题，导致数据处理错乱，而且不容易发现。为此，我们项目定了一些基本原则。

UI 操作和 DataSource 的操作一定在主线程。
DB 操作、日志记录、网络回调都在各自的固定线程。
不同业务，可以通过创建队列保证数据一致性。例如，想法列表的数据加载、书籍章节下载、书架加载等。
合理的线程分配，最终目的就是保证主线程尽量少的处理非UI操作，同时控制整个App的子线程数量在合理的范围内。


3.预处理和延时加载
预处理，是将初次显示需要耗费大量线程时间的操作，提前放到后台线程进行计算，再将结果数据拿来显示。

延时加载，是指首先加载当前必须的可视内容，在稍后一段时间内或特定事件时，再触发其他内容的加载。这种方式可以很有效的提升界面绘制速度，使体验更加流畅。（UITableView 就是最典型的例子）

这两种方法都是在资源比较紧张的情况下，优先处理马上要用到的数据，同时尽可能提前加载即将要用到的数据。在微信读书中阅读的排版是优先级最高的，所在在阅读过程中会预处理下一页、下一章的排版，同时可能会延时加载阅读相关的其它数据（如想法、划线、书签等）。


4.缓存
cache可能是所有性能优化中最常用的手段，但也是我们极不推荐的手段。cache建立的成本低，见效快，但是带来维护的成本却很高。如果一定要用，也请谨慎使用，并注意以下几点：

并发访问 cache 时，数据一致性问题。
cache 线程安全问题，防止一边修改一边遍历的 crash。
cache 查找时性能问题。
cache 的释放与重建，避免占用空间无限扩大，同时释放的粒度也要依实际需求而定。


5.使用正确的API
使用正确的 API，是指在满足业务的同时，能够选择性能更优的API。

选择合适的容器;
了解 imageNamed: 与 imageWithContentsOfFile:的差异(imageNamed: 适用于会重复加载的小图片，因为系统会自动缓存加载的图片，imageWithContentsOfFile: 仅加载图片)
缓存 NSDateFormatter 的结果。
寻找 (NSDate *)dateFromString:(NSString )string 的替换品。
1
2
3
4
5
6
7
//#include <time.h>
time_t t;
struct tm tm;
strptime([iso8601String cStringUsingEncoding:NSUTF8StringEncoding], "%Y-%m-%dT%H:%M:%S%z", &tm);
tm.tm_isdst = -1;
t = mktime(&tm);
[NSDate dateWithTimeIntervalSince1970:t + [[NSTimeZone localTimeZone] secondsFromGMT]];
不要随意使用 NSLog().

当试图获取磁盘中一个文件的属性信息时，使用 [NSFileManager attributesOfItemAtPath:error:] 会浪费大量时间读取可能根本不需要的附加属性。这时可以使用 stat 代替 NSFileManager，直接获取文件属性：

1
2
3
4
5
6
7
8
9
#import <sys/stat.h>
struct stat statbuf;
const char *cpath = [filePath fileSystemRepresentation];
if (cpath && stat(cpath, &statbuf) == 0) {
    NSNumber *fileSize = [NSNumber numberWithUnsignedLongLong:statbuf.st_size];
    NSDate *modificationDate = [NSDate dateWithTimeIntervalSince1970:statbuf.st_mtime];
    NSDate *creationDate = [NSDate dateWithTimeIntervalSince1970:statbuf.st_ctime];
    // etc
}



3. UI / DataSource主线程检测工具。
该工具是为了保证所有的UI的操作和 DataSource 操作一定是在主线程进行，同样是由tower同学贡献。实现原理是通过 hook UIView 的 -setNeedsLayout，-setNeedsDisplay，-setNeedsDisplayInRect 三个方法，确保它们都是在主线程执行。子线程操作UI可能会引起什么问题，苹果说得并不清楚，实际开发中我们遇到几种神奇的问题似乎都是跟这个有关。

app 突然丢动画，似乎 iOS 系统也有这个 bug。虽然没有确切的证据，但使用这个工具，改完所有的问题后，bug 也好了(不止一次是这样)。

UI 操作偶尔响应特别慢，从代码看没有任何耗时操作，只是简单的 push 某个 controller。

莫名的 crash，这当然是因为 UI 操作非线程安全引起的。

更多时候，子线程操作 UI 也并不一定会发生什么问题，也正因为不知道会发生什么，所以更需要我们警惕，这个工具替我们扫除了这些隐患。虽然，苹果表示，现在部分的 UI 操作也已经是线程安全了，但毕竟大部分还不是。DataSource 的监测是因为我们业务定下的原则，保证列表 DataSource 的线程安全。

针对低端机型，去掉了某些动画，交互更加流畅。
