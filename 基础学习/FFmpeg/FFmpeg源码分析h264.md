## libavcodec\h264_decode_init.c

##### h264_decoder

```c
AVCodec ff_h264_decoder = {
.name                  = "h264",
.long_name             = NULL_IF_CONFIG_SMALL("H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10"),
.type                  = AVMEDIA_TYPE_VIDEO,
.id                    = AV_CODEC_ID_H264,
.priv_data_size        = sizeof(H264Context),
.init                  = h264_decode_init,
.close                 = h264_decode_end,
.decode                = h264_decode_frame,
.capabilities          = /*AV_CODEC_CAP_DRAW_HORIZ_BAND |*/ AV_CODEC_CAP_DR1 |
                         AV_CODEC_CAP_DELAY | AV_CODEC_CAP_SLICE_THREADS |
                         AV_CODEC_CAP_FRAME_THREADS,
.caps_internal         = FF_CODEC_CAP_INIT_THREADSAFE | FF_CODEC_CAP_EXPORTS_CROPPING,
.flush                 = flush_dpb,
.init_thread_copy      = ONLY_IF_THREADS_ENABLED(decode_init_thread_copy),
.update_thread_context = ONLY_IF_THREADS_ENABLED(ff_h264_update_thread_context),
.profiles              = NULL_IF_CONFIG_SMALL(ff_h264_profiles),
.priv_class            = &h264_class,
};
```



h264_decode_init -> h264_decode_frame - > h264_decode_end



1. h264_decode_init

   ​

2. h264_decode_frame

3. h264_decode_end

   ​