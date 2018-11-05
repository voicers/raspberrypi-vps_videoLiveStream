# raspberrypi-vps_videoLiveStream

（非rtmp）用树莓派与云主机（以阿里云为例）搭建直播服务器（1）

采取方案：udp推流至阿里云，在阿里云上转流、直播（使用ffmpeg与ffserver）
直播方案：
                 
                 a.视频封装：asf，测试推流格式：mjpeg/h264等，支持原生浏览器平台：windows
                 
                 b.视频封装：swf，测试推流格式：mjpeg，支持原生浏览器平台：win/mac
                 
                 c.视频封装：mjpeg，测试推流格式：mjpeg，支持原生浏览器：全平台，视频质量：中
                
                d.视频封装：hls/m3u8，推流格式h264，全平台

其中方案a测试中有2-3秒延时，方案b在mac下似乎实现了准实时的较好直播，
方案c可控性一般，方案d有5-10s延时


准备工作：
 
 在树莓派上 sudo apt-get install ffmpeg

 在阿里云 sudo apt-get install ffmpeg
 
 在防火墙打开阿里云端口8090、9000、9001

编写配置文件：

cd /the/path/to/your/config/folder

cp /etc/ffserver.conf .

nano ffserver.conf

下面来编写conf文件

在《stream test1.conf》与《/stream》之间所有没有加#的行前加#

在example streams 下面的类型里选择你想要的推流格式，并把该类所有行前面的#去掉，把你不想要的格式行前加#

配置直播流：根据你的要求配置直播流（关键），可以参考前面他写的，例如只推视频流：加上NoAudio，并在所有含audio的行前加#，一定要设置帧率VideoFrameRate 你想要的帧率。这个一定要写，默认帧率为2fps或5fps，我就是这个原因搞了一天，即使你在这一行前面加#他依旧会使用默认帧率

支持推两种封装格式，但最好采用同一种编码形式，在配置文件里留两种输出即可


之后运行

ffserver -f your/config/file

则服务器开始运行

在树莓派推流：

这里使用udp推流，推usb摄像头，推送mjpeg视频（无音频）流为例

ffmpeg -re -i /dev/video0 -an -vcodec mjpeg -f mjpeg udp://yourIpAddress:9000

在服务器采用ffmpeg转流：

ffmpeg -f mjpeg -i udp://yourIpAddress:9000 udp://0.0.0.0:8090/feed1.ffm

这时，你在yourIpAddress:8090/stat.html便可以看见你的所有视频文件地址

在html标签中插入所有链接，播放器即可

而mjpeg格式可以作为图片插入html5中

对于hls流，直接生成m3u8文件即可，树莓派可以通过udp推流至服务器（甚至ftp貌似也可以，
推荐原始编码就用h264），可以不需要ffserver，在服务器上直接ffmpeg -i XXX XXX.m3u8，搭配apache/django将m3u8打开或插入网页即可。

