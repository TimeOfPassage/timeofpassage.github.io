# #ffmpeg操作

# FFmpeg

官网: [https://www.ffmpeg.org/](https://www.ffmpeg.org/ "https://www.ffmpeg.org/")

> 参考：https://wangchujiang.com/reference/docs/ffmpeg.html

# 基本命令

```bash
ffmpeg [global_options] {[input_file_options] -i input_file} ... {[output_file_options] output_file} ...

ffmpeg [全局选项] {[输入文件选项] -i 输入文件/url地址}... {[输出文件选项] 输出文件/url地址}...

#参数 -i 用来读取任意数量的输入文件(可以是正常文件/管道/网络流/抓取设备等)，
#写入任意数量的被声明对象为一个简单的输出文件名的文件。
#在命令行的任何不能被解释的选项都被作为输出文件。
```

# 帮助命令

```bash
Getting help:
    -h -- print basic options    #打印基本选项
    -h long -- print more options   #打印更多选项
    -h full -- print all options (including all format and codec specific options, very long)  
                  #打印所有选项（包括所有格式和编解码器特定选项，非常长）
    -h type=name -- print all options for the named decoder/encoder/demuxer/muxer/filter/bsf
                             #打印指定解码器/编码器/解复用器/muxer/filter的 所有选项
    See man ffmpeg for detailed description of the options.  #有关选项的详细说明，请参见man ffmpeg。

Print help / information / capabilities: #打印帮助/信息/功能
-L  show license   #显示许可证
-h  topic show help   #主题显示帮助
-?  topic show help   #主题显示帮助
-help  topic show help   #主题显示帮助
--help  topic show help   #主题显示帮助
```

‍

# 功能显示

```bash
-version    show version   #显示版本
-buildconf    show build configuration   #显示构建配置
-formats    show available formats   #显示可用的格式/设备

-muxers    show available muxers   #显示可用的复用器
-demuxers    show available demuxers   #显示可用的解复用器
-devices    show available devices   #显示可用的设备

-codecs    show available codecs   #显示可用的编解码器
-decoders    show available decoders   #显示可用的解码器
-encoders    show available encoders   #显示可用的编码器

-bsfs    show available bit stream filters   #显示可用的位流过滤器
-protocols    show available protocols   #显示可用的协议
-filters    show available filters   #显示可用的过滤器

-pix_fmts    show available pixel formats   #显示可用的像素格式
-layouts    show standard channel layouts   #显示标准通道布局
-sample_fmts    show available audio sample formats   #显示可用的音频采样格式
-colors    show available color names   #显示识别的颜色名称

-sources device    list sources of the input device   #设备列表 输入设备的源
-sinks device    list sinks of the output device   #设备列表 输出设备的接收器
-hwaccels    show available HW acceleration methods   #显示可用的硬件加速方法
```

# **全局选项**

**全局选项 [global_options]：（影响整个程序，而不仅仅是一个文件）**

```bash
Global options (affect whole program instead of just one file:
-loglevel  loglevel    set logging level   #设置日志记录级别
-v  loglevel    set logging level   #设置日志记录级别
-report    generate a report   #生成报告
-max_alloc  bytes    set maximum size of a single allocated block   #设置单个已分配块的最大大小
-y    overwrite output files   #覆盖输出文件
-n    never overwrite output files   #永不覆盖输出文件
-ignore_unknown    Ignore unknown stream types   #忽略未知的媒体流类型
-filter_threads    number of non-complex filter threads   #非复杂过滤器线程的数量
-filter_complex_threads    number of threads for -filter_complex   #-filter_complex的线程数
-stats    print progress report during encoding   #在编码期间打印进度报告
-max_error_rate    maximum error rate  ratio of errors (0.0: no errors, 1.0: 100% errors) above which ffmpeg returns an error instead of success.  
                       #错误率（0.0：无错误，1.0：100％错误）以上报错。
-bits_per_raw_sample number    set the number of bits per raw sample   #设置每个原始样本的位数
-vol volume    change audio volume (256=normal)   #音量改变音量（256 =正常）
```

# **文件主选项**

**文件主选项 [input_file_options] / [output_file_options]**

```bash
Per-file main options:
-f  fmt              force format   #指定格式
-c  codec            codec name   #设置编解码器
-codec  codec        codec name   #设置编解码器
-pre  preset         preset name   #设置预处理集
-map_metadata outfile[,metadata]:infile[,metadata]     set metadata information of outfile from infile   #输出文件的元数据信息

-t  duration         record or transcode "duration" seconds of audio/video   #设置音/视频的录制或转码的“持续时间”秒 。
-to  time_stop       record or transcode stop time   #设置录制或转码的停止时间，时间格式：HOURS:MM:SS.MICROSECONDS。
                            #注: -to(时间格式)和 -t(秒数)选项都是表述持续时间，只是格式不同。

-fs  limit_size      set the limit file size in bytes   #设置文件大小上限，单位字节
-ss  time_off       set the start time offset   #设置开始时间，开始点后多久后正常工作，时间格式：X秒 或 HOURS:MM:SS.MICROSECONDS
-sseof  time_off       set the start time offset relative to EOF   #设置结束时间偏移量
-seek_timestamp       enable/disable seeking by timestamp with -ss   #启用或禁止使用-ss参数去根据时间戳拖动
-timestamp  time       set the recording timestamp ('now' to set the current time)   #设置录制的时间戳(now表示现在、当前时间)
-metadata  string=string       add metadata   #添加元数据
-program  title=string:st=number...       add program with specified streams   #给指定的流添加节目
-target  type       specify target file type ("vcd", "svcd", "dvd", "dv" or "dv50" with optional prefixes "pal-", "ntsc-" or "film-")
                      #设置目标文件类型(vcd,svcd,dvd) ，所有的格式选项（比特率，编解码以及缓冲区大小）自动设置。
                      #输入格式：ffmpeg -i input.avi -target vcd ouput.mpg
-apad    audio pad   #音频补齐
-frames  number       set the number of frames to output   #设置输出帧数
-filter  filter_graph       set stream filtergraph   #设置流的滤镜图
-filter_script  filename       read stream filtergraph description from a file   #从一个文件读取流的滤镜图的描述（以文件的形式添加滤镜图）
-reinit_filter       reinit filtergraph on input parameter changes   #当输入参数变化时重新初始化滤镜图
-discard       discard   #丢弃
-disposition       disposition   #配置
```

**文件主选项**-其他：

```bash
-re：代表按照时间戳读取或发送数据，尤其在作为推流工具的时候一定要加上该参数，否则ffpmeg会按照最高速率向流媒体不停的发送数据。
-map：指定输出文件的流映射关系。例如：“-map 1:0 -map 1:1”要求按照第二个输入的文件的第一个流和第二个流写入输出文件。如果没有设置此项，则ffpmeg采用默认的映射关系。
-title string    设置标题
-author string    设置作者
-copyright string   设置版权
-comment string   设置评论
-hq   激活高质量设置
-itsoffset offset    设置以秒为基准的时间偏移，该选项影响所有后面的输入文件。该偏移被加到输入文件的时戳，定义一个正偏移意味着相应的流被延迟了 offset秒。 [-]hh:mm:ss[.xxx]的格式也支持
```

视频参数：

```bash
Video options:
-vframes  number     set the number of video frames to output   #设置要输出的视频帧数
-r  rate     set frame rate (Hz value, fraction or abbreviation)   #设置帧速率（Hz值，分数或缩写）
-s  size     set frame size (WxH or abbreviation)   #指定分辨率（WxH或缩写）（320x240）
-aspect  aspect     set aspect ratio (4:3, 16:9 or 1.3333, 1.7777)   #设置视频长宽比（4:3、16:9或1.33333、1.77777）
-bits_per_raw_sample  number     set the number of bits per raw sample   #设置每个原始样本的位数
-vn     disable video   #取消视频输出
-vcodec  codec     force video codec ('copy' to copy stream)   #指定使用视频编解码器（“复制”到复制流）
-c codeccodec name编解码器名称
-vcodec codecforce video codec ('copy' to copy stream)强制视频编解码器（“复制”以复制流）
-acodec codecforce audio codec ('copy' to copy stream)强制音频编解码器（“复制”以复制流）
-scodec codecforce subtitle codec ('copy' to copy stream)强制字幕编解码器（“复制”以复制流）
增加了 -c copy 直接复制. 参照上面帮助信息, 也可以分开来写 -vcodec copy -acodec copy.

(也有后面这样的写法: -c:v copy -c:a copy 但这种写法之对部分文件有效, 对于美剧原始文件(包含更多的元数据,流数据)会报错.)
快速剪裁视频前100秒, 这种写法对于国内翻译网站编辑过的文件有效.
$ ffmpeg -ss 00:00:00 -i Devs.mp4 -t 100 -c:v copy -c:a copy Deus.mp4
-timecode  hh:mm:ss[:;.]ff     set initial TimeCode value.   #设置初始时间值
-pass  n     select the pass number (1 to 3)   #选择通行证号码（1到3）
-vf  filter_graph     set video filters   #设置视频过滤器
-ab  bitrate     audio bitrate (please use -b:a)   #指定音频比特率（bit/s） -b:a
-b  bitrate     video bitrate (please use -b:v)   #指定视频比特率（bit/s）-b:v ffmpeg默认采用的是VBR，若指定的该参数，则使用平均比特率。
-dn     disable data   #禁止数据缓存
```

视频参数-其他：

```bash
-bitexact：使用标准比特率。
-vb：指定视频比特率（bit/s）
-croptop size：设置顶部切除尺寸（in pixels）
-cropleft size：设置左切除尺寸（in pixels）
-cropbottom size：设置地步切除尺寸（in pixels）
-cropright size：设置右切除尺寸（in pixels）
-padtop size：设置顶部补齐尺寸（in pixels）
-padleft size：设置左补齐尺寸（in pixels）
-padbottom size：设置地步补齐尺寸（in pixels）
-padright size：设置右补齐尺寸（in pixels）
-padcolor color：设置补齐颜色
```

音频参数：

```bash
Audio options:
-aframes number     set the number of audio frames to output   #设置输出音频帧数
-aq quality         set audio quality (codec-specific)   #设置音频质量
-ar rate            set audio sampling rate (in Hz)   #设置音频采样率（Hz）
-ac channels        set number of audio channels   #设置声道数，1为单声道，2为立体声
-an                 disable audio   #取消音频输出
-acodec codec       force audio codec ('copy' to copy stream)   #指定音频编解码器（“复制”到复制流）
-vol volume         change audio volume (256=normal)   #设置录制音量大小
-af filter_graph    set audio filters   #设置音频滤镜
```

音频参数-其他

```bash
-ab：
设置比特率（bit/s），对于MP3的格式，
想要听到较高品质的声音，建议设置160Kbit/s（单声道80Kbit/s）以上。
```

字幕参数：

```bash
Subtitle options:
-s size             set frame size (WxH or abbreviation)    #设置帧数的大小
-sn                 disable subtitle    #禁止字幕
-scodec codec       force subtitle codec ('copy' to copy stream) #设置字幕的编解码器，copy表示使用输入源
-stag fourcc/tag    force subtitle tag/fourcc    #设置字幕tag
-fix_sub_duration   fix subtitles duration    #修改、纠正字幕的时间
-canvas_size size   set canvas size (WxH or abbreviation)    #设置画布大小
-spre preset        set the subtitle options to the indicated preset    #设置字幕选项的预设(预处理集)
```

高级视频参数：

```bash
-g gop_size 设置图像组大小
-intra 仅适用帧内编码
-qscale q 使用固定的视频量化标度(VBR)
-qmin q 最小视频量化标度(VBR)
-qmax q 最大视频量化标度(VBR)
-qdiff q 量化标度间最大偏差 (VBR)
-qblur blur 视频量化标度柔化(VBR)
-qcomp compression 视频量化标度压缩(VBR)
-rc_init_cplx complexity 一遍编码的初始复杂度
-b_qfactor factor 在p和b帧间的qp因子
-i_qfactor factor 在p和i帧间的qp因子
-b_qoffset offset 在p和b帧间的qp偏差
-i_qoffset offset 在p和i帧间的qp偏差
-rc_eq equation 设置码率控制方程 默认tex^qComp
-rc_override override 特定间隔下的速率控制重载
-me method 设置运动估计的方法 可用方法有 zero phods log x1 epzs(缺省) full
-dct_algo algo 设置dct的算法 可用的有 0 FF_DCT_AUTO 缺省的DCT 1 FF_DCT_FASTINT 2 FF_DCT_INT 3 FF_DCT_MMX 4 FF_DCT_MLIB 5 FF_DCT_ALTIVEC
-idct_algo algo 设置idct算法。可用的有 0 FF_IDCT_AUTO 缺省的IDCT 1 FF_IDCT_INT 2 FF_IDCT_SIMPLE 3 FF_IDCT_SIMPLEMMX 4 FF_IDCT_LIBMPEG2MMX 5 FF_IDCT_PS2 6 FF_IDCT_MLIB 7 FF_IDCT_ARM 8 FF_IDCT_ALTIVEC 9 FF_IDCT_SH4 10 FF_IDCT_SIMPLEARM
-er n 设置错误残留为n 1 FF_ER_CAREFULL 缺省 2 FF_ER_COMPLIANT 3 FF_ER_AGGRESSIVE 4 FF_ER_VERY_AGGRESSIVE
-ec bit_mask 设置错误掩蔽为bit_mask,该值为如下值的位掩码 1 FF_EC_GUESS_MVS (default=enabled) 2 FF_EC_DEBLOCK (default=enabled)
-bf frames 使用frames B 帧，支持mpeg1,mpeg2,mpeg4
-mbd mode 宏块决策 0 FF_MB_DECISION_SIMPLE 使用mb_cmp 1 FF_MB_DECISION_BITS 2 FF_MB_DECISION_RD
-4mv 使用4个运动矢量 仅用于mpeg4
-part 使用数据划分 仅用于mpeg4
-bug param 绕过没有被自动监测到编码器的问题
-strict strictness 跟标准的严格性
-aic 使能高级帧内编码 h263+
-umv 使能无限运动矢量 h263+
-deinterlace 不采用交织方法
-interlace 强迫交织法编码仅对mpeg2和mpeg4有效。当你的输入是交织的并且你想要保持交织以最小图像损失的时候采用该选项。可选的方法是不交织，但是损失更大
-psnr 计算压缩帧的psnr
-vstats 输出视频编码统计到vstats_hhmmss.log
-vhook module 插入视频处理模块 module 包括了模块名和参数，用空格分开
```

音频/视频捕获选项

```bash
-vd device 设置视频捕获设备。比如/dev/video0
-vc channel 设置视频捕获通道 DV1394专用
-tvstd standard 设置电视标准 NTSC PAL(SECAM)
-dv1394 设置DV1394捕获
-av device 设置音频设备 比如/dev/dsp
```

高级选项

```bash
-map file:stream 设置输入流映射
-debug 打印特定调试信息
-benchmark 为基准测试加入时间
-hex 倾倒每一个输入包
-bitexact 仅使用位精确算法 用于编解码测试
-ps size 设置包大小，以bits为单位
-re 以本地帧频读数据，主要用于模拟捕获设备
-loop 循环输入流（只工作于图像流，用于ffserver测试）
```

‍

# 实际命令

## flac转mp3

```bash
# 同时会将flac文件的Vorbis注释转换为mp3的ID3v2元数据
ffmpeg -i input.flac -ab 320k -map_metadata 0 -id3v2_version 3 output.mp3
```

## mp4提取mp3

```bash
ffmpeg -i  input.mp4  -vn  output.mp3

ffmpeg.exe  -i  input.mp3 -ss 00:00:00  -t  00:01:00  output.mp3
```

## mp3音频截取

```bash
ffmpeg -i input.mp3 -ss 00:00:00 -t  00:01:00 output.mp3
```

## 去除静态水印

```bash
ffmpeg -i input.mp4 -b:v 3170k -vf  "delogo=x=1:y=1:w=100:h=30:show=0" output.mp4

#-b:v 3170k 是设置视频的码率，可不加。
#-vf  "delogo=x=1:y=1:w=100:h=30:show=0" 给视频添加一个类似马赛克的滤镜效果，大小是以视频左上角为（x=1，y=1）的坐标，w宽为100，h高为30的滤镜
#如果 show=1 就会有一个绿框，show=0为不可见。
```

## 添加水印

```java
ffmpeg -y -i input.mp4 -i input.png 
-filter_complex 
	[0:v]scale=iw:ih[outv0];
	[1:0]scale=0.0:0.0[outv1];
	[outv0][outv1]overlay=0:0 
-preset superfast result.mp4
```

1. ​`-y`​: 覆盖输出文件而无需确认。
2. ​`-i input.mp4`​: 指定输入视频文件为 "input.mp4"。
3. ​`-i input.png`​: 指定输入图片文件为 "input.png"。
4. ​`-filter_complex [...]`​: 这是用于处理音视频的复杂滤镜图。下面是具体的解释：

    * ​`[0:v]scale=iw:ih[outv0]`​: 对第一个输入文件（视频文件）进行处理，将视频的宽度和高度保持不变，结果保存为 "outv0"。
    * ​`[1:0]scale=0.0:0.0[outv1]`​: 对第二个输入文件（图片文件的视频流）进行处理，将其缩放为零大小，结果保存为 "outv1"。
    * ​`[outv0][outv1]overlay=0:0`​: 将两个处理后的视频流叠加在一起，位置为 (0, 0)。
5. ​`-preset superfast`​: 使用视频编码器的超快速度预设。
6. ​`result.mp4`​: 指定输出文件名为 "result.mp4"。

## 添加背景音乐（支持调节原音和配乐的音量）

```bash
ffmpeg -y -i input.mp4 -i input.mp3 
-filter_complex 
	[0:a]aformat=sample_fmts=fltp:sample_rates=44100:channel_layouts=stereo,volume=0.2[a0];
	[1:a]aformat=sample_fmts=fltp:sample_rates=44100:channel_layouts=stereo,volume=1[a1];
	[a0][a1]amix=inputs=2:duration=first[aout] -map [aout] 
-ac 2 -c:v copy -map 0:v:0 -preset superfast result.mp4
```

1. ​`-y`​: 覆盖输出文件而无需确认。
2. ​`-i input.mp4`​: 指定输入视频文件为 "input.mp4"。
3. ​`-i input.mp3`​: 指定输入音频文件为 "input.mp3"。
4. ​`-filter_complex [...]`​: 这是用于处理音频和视频的滤镜图。下面是具体的解释：

    * ​`[0:a]aformat=sample_fmts=fltp:sample_rates=44100:channel_layouts=stereo,volume=0.2[a0]`​: 对第一个输入文件（视频的音频流）进行处理，设置采样格式为 fltp，采样率为 44100 Hz，声道布局为立体声，音量调整为 0.2，并将结果保存为 "a0"。
    * ​`[1:a]aformat=sample_fmts=fltp:sample_rates=44100:channel_layouts=stereo,volume=1[a1]`​: 对第二个输入文件（音频文件）进行处理，设置相同的音频属性，但音量调整为 1，并将结果保存为 "a1"。
    * ​`[a0][a1]amix=inputs=2:duration=first[aout]`​: 将 "a0" 和 "a1" 混合在一起，设置输入数量为 2，以第一个输入的持续时间为准，并将结果保存为 "aout"。
5. ​`-map [aout]`​: 选择之前处理得到的混合音频流作为输出音频流。
6. ​`-ac 2`​: 设置输出音频流的声道数为 2（立体声）。
7. ​`-c:v copy`​: 复制输入视频的视频编码方式，以避免重新编码视频流。
8. ​`-map 0:v:0`​: 选择输入视频的第一个视频流作为输出视频流。
9. ​`-preset superfast`​: 使用视频编码器的超快速度预设。
10. ​`result.mp4`​: 指定输出文件名为 "result.mp4"。

# Gif转视频

```bash
ffmpeg -y -i input.gif -pix_fmt yuv420p -preset superfast result.mp4
```

## 视频转Gif

```java
ffmpeg -y -ss 0 -t 7 -i input.mp4 -r 5 -s 280x606 -preset superfast result.gif
```

‍
