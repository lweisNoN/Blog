FFMPeg 的一次编解码过程如下,

    av_log_set_callback(FFLog);
    av_register_all();
    avformat_network_init();
    avformat_alloc_context();
    avformat_open_input();


1. 注册复用器，编码器，网络协议等。
2. 初始化数据封装（FLV/MKV/RMVB/MP4等）结构体，分配相关内存。
3. 通过文件名或者文件头识别输入输出协议。
4. 通过识别出的协议的 read，open 等方法读写数据

以下为详细解析

## 编、解码器的注册和查找

* #### avcodec_register_all [libavcodec/allcodecs.c](https://github.com/FFmpeg/FFmpeg/blob/master/libavcodec/allcodecs.c)

```
register_all() {
  /* hardware accelerators，硬件加速器 */
  REGISTER_HWACCEL()	//eg. REGISTER_HWACCEL(H264_CUVID, h264_cuvid);
  
  /* 包含编码器和解码器 */
  REGISTER_ENCDEC()		//eg. REGISTER_ENCDEC (H263, h263);
  
  /* 编码器 */
  REGISTER_ENCODER()	//REGISTER_ENCODER(LJPEG, ljpeg);
  
  /* 解码器 */
  REGISTER_DECODER()	//eg. REGISTER_DECODER(H264, h264);
  
  /* parsers */
  REGISTER_PARSER()		//eg. REGISTER_PARSER(AAC, aac);
}
	
```

```
#define REGISTER_HWACCEL(X, x)                                          \
    {                                                                   \
        extern AVHWAccel ff_##x##_hwaccel;                              \
        if (CONFIG_##X##_HWACCEL)                                       \
            av_register_hwaccel(&ff_##x##_hwaccel);                     \
    }

#define REGISTER_ENCODER(X, x)                                          \
    {                                                                   \
        extern AVCodec ff_##x##_encoder;                                \
        if (CONFIG_##X##_ENCODER)                                       \
            avcodec_register(&ff_##x##_encoder);                        \
    }

#define REGISTER_DECODER(X, x)                                          \
    {                                                                   \
        extern AVCodec ff_##x##_decoder;                                \
        if (CONFIG_##X##_DECODER)                                       \
            avcodec_register(&ff_##x##_decoder);                        \
    }

#define REGISTER_ENCDEC(X, x) REGISTER_ENCODER(X, x); REGISTER_DECODER(X, x)

#define REGISTER_PARSER(X, x)                                           \
    {                                                                   \
        extern AVCodecParser ff_##x##_parser;                           \
        if (CONFIG_##X##_PARSER)                                        \
            av_register_codec_parser(&ff_##x##_parser);                 \
    }
```

* #### avcodec_register [utils.c](https://github.com/FFmpeg/FFmpeg/blob/master/libavcodec/utils.c)

```
av_cold void avcodec_register(AVCodec *codec)
{
    AVCodec **p;
    avcodec_init();
    p = last_avcodec;
    codec->next = NULL;

    while(*p || avpriv_atomic_ptr_cas((void * volatile *)p, NULL, codec))
        p = &(*p)->next;
    last_avcodec = &codec->next;

    if (codec->init_static_data)
        codec->init_static_data(codec);
}
```

`last_avcodec` 是 format 链表的最后指针， `avcodec_register` 函数从这个静态指针开始遍历 format 链表，将新 register 的 AVcodec （e.g.: h.264)，链入链表的最后一位。

需要注意的是`avpriv_atomic_ptr_cas`函数中，第一个参数 p 的 next 等于第二个参数传入的 NULL 时，p->next 会被赋值为第三个参数 codec。

* #### avcodec_find_encoder, avcodec_find_encoder_by_name, avcodec_find_decoder, avcodec_find_decoder_by_name  [utils.c](https://github.com/FFmpeg/FFmpeg/blob/master/libavcodec/utils.c)

解码器作为链表方式存储，通过上述链表查找对应的解码器。







