---
layout: article
title: NDK r21d + FFmpeg 4.2 编写脚本文件 & 编译生成 Android 所需的库
date: 2020-09-22 03:20:30
tags:
categories: 
copyright: true
---

# **Reference**
* [FFmpeg](http://ffmpeg.org/ "http://ffmpeg.org/")
* [FFMPEG 配置选项详细说明](https://blog.csdn.net/z2066411585/article/details/81239446 "https://blog.csdn.net/z2066411585/article/details/81239446")
* [FFmpeg编译4.1.4并移植到Android](https://zhuanlan.zhihu.com/p/76462890 "https://zhuanlan.zhihu.com/p/76462890")

---

# **下载 FFmpeg 源码**
```shell
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
```

---

# **编写脚本文件**

## **查看文件目录**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/1.png)

## **查看支持的命令**
```shell
./configure --help
```
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/2.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/3.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/4.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/5.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/6.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/7.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/8.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/9.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/10.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/11.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/12.png)

### **decoders**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/13.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/14.png)

decoders||||||||
--|--
aac                     |amrwb                   |dst                     |hnm4_video              |mp1                     |pcm_f32le               |rv30                    |vc1_v4l2m2m
aac_at                  |amv                     |dvaudio                 |hq_hqa                  |mp1_at                  |pcm_f64be               |rv40                    |vc1image
aac_fixed               |anm                     |dvbsub                  |hqx                     |mp1float                |pcm_f64le               |s302m                   |vcr1
aac_latm                |ansi                    |dvdsub                  |huffyuv                 |mp2                     |pcm_lxf                 |sami                    |vmdaudio
aasc                    |ape                     |dvvideo                 |hymt                    |mp2_at                  |pcm_mulaw               |sanm                    |vmdvideo
ac3                     |apng                    |dxa                     |iac                     |mp2float                |pcm_mulaw_at            |sbc                     |vmnc
ac3_at                  |aptx                    |dxtory                  |idcin                   |mp3                     |pcm_s16be               |scpr                    |vorbis
ac3_fixed               |aptx_hd                 |dxv                     |idf                     |mp3_at                  |pcm_s16be_planar        |screenpresso            |vp3
acelp_kelvin            |arbc                    |eac3                    |iff_ilbm                |mp3adu                  |pcm_s16le               |sdx2_dpcm               |vp4
adpcm_4xm               |ass                     |eac3_at                 |ilbc                    |mp3adufloat             |pcm_s16le_planar        |sgi                     |vp5
adpcm_adx               |asv1                    |eacmv                   |ilbc_at                 |mp3float                |pcm_s24be               |sgirle                  |vp6
adpcm_afc               |asv2                    |eamad                   |imc                     |mp3on4                  |pcm_s24daud             |sheervideo              |vp6a
adpcm_agm               |atrac1                  |eatgq                   |imm4                    |mp3on4float             |pcm_s24le               |shorten                 |vp6f
adpcm_aica              |atrac3                  |eatgv                   |imm5                    |mpc7                    |pcm_s24le_planar        |sipr                    |vp7
adpcm_argo              |atrac3al                |eatqi                   |indeo2                  |mpc8                    |pcm_s32be               |siren                   |vp8
adpcm_ct                |atrac3p                 |eightbps                |indeo3                  |mpeg1_cuvid             |pcm_s32le               |smackaud                |vp8_cuvid
adpcm_dtk               |atrac3pal               |eightsvx_exp            |indeo4                  |mpeg1_v4l2m2m           |pcm_s32le_planar        |smacker                 |vp8_mediacodec
adpcm_ea                |atrac9                  |eightsvx_fib            |indeo5                  |mpeg1video              |pcm_s64be               |smc                     |vp8_qsv
adpcm_ea_maxis_xa       |aura                    |escape124               |interplay_acm           |mpeg2_crystalhd         |pcm_s64le               |smvjpeg                 |vp8_rkmpp
adpcm_ea_r1             |aura2                   |escape130               |interplay_dpcm          |mpeg2_cuvid             |pcm_s8                  |snow                    |vp8_v4l2m2m
adpcm_ea_r2             |av1                     |evrc                    |interplay_video         |mpeg2_mediacodec        |pcm_s8_planar           |sol_dpcm                |vp9
adpcm_ea_r3             |avrn                    |exr                     |jacosub                 |mpeg2_mmal              |pcm_u16be               |sonic                   |vp9_cuvid
adpcm_ea_xas            |avrp                    |fastaudio               |jpeg2000                |mpeg2_qsv               |pcm_u16le               |sp5x                    |vp9_mediacodec
adpcm_g722              |avs                     |ffv1                    |jpegls                  |mpeg2_v4l2m2m           |pcm_u24be               |speedhq                 |vp9_qsv
adpcm_g726              |avui                    |ffvhuff                 |jv                      |mpeg2video              |pcm_u24le               |srgc                    |vp9_rkmpp
adpcm_g726le            |ayuv                    |ffwavesynth             |kgv1                    |mpeg4                   |pcm_u32be               |srt                     |vp9_v4l2m2m
adpcm_ima_alp           |bethsoftvid             |fic                     |kmvc                    |mpeg4_crystalhd         |pcm_u32le               |ssa                     |vplayer
adpcm_ima_amv           |bfi                     |fits                    |lagarith                |mpeg4_cuvid             |pcm_u8                  |stl                     |vqa
adpcm_ima_apc           |bink                    |flac                    |libaom_av1              |mpeg4_mediacodec        |pcm_vidc                |subrip                  |wavpack
adpcm_ima_apm           |binkaudio_dct           |flashsv                 |libaribb24              |mpeg4_mmal              |pcx                     |subviewer               |wcmv
adpcm_ima_cunning       |binkaudio_rdft          |flashsv2                |libcelt                 |mpeg4_v4l2m2m           |pfm                     |subviewer1              |webp
adpcm_ima_dat4          |bintext                 |flic                    |libcodec2               |mpegvideo               |pgm                     |sunrast                 |webvtt
adpcm_ima_dk3           |bitpacked               |flv                     |libdav1d                |mpl2                    |pgmyuv                  |svq1                    |wmalossless
adpcm_ima_dk4           |bmp                     |fmvc                    |libdavs2                |msa1                    |pgssub                  |svq3                    |wmapro
adpcm_ima_ea_eacs       |bmv_audio               |fourxm                  |libfdk_aac              |mscc                    |pgx                     |tak                     |wmav1
adpcm_ima_ea_sead       |bmv_video               |fraps                   |libgsm                  |msmpeg4_crystalhd       |photocd                 |targa                   |wmav2
adpcm_ima_iss           |brender_pix             |frwu                    |libgsm_ms               |msmpeg4v1               |pictor                  |targa_y216              |wmavoice
adpcm_ima_moflex        |c93                     |g2m                     |libilbc                 |msmpeg4v2               |pixlet                  |tdsc                    |wmv1
adpcm_ima_mtf           |cavs                    |g723_1                  |libopencore_amrnb       |msmpeg4v3               |pjs                     |text                    |wmv2
adpcm_ima_oki           |ccaption                |g729                    |libopencore_amrwb       |msrle                   |png                     |theora                  |wmv3
adpcm_ima_qt            |cdgraphics              |gdv                     |libopenh264             |mss1                    |ppm                     |thp                     |wmv3_crystalhd
adpcm_ima_qt_at         |cdtoons                 |gif                     |libopenjpeg             |mss2                    |prores                  |tiertexseqvideo         |wmv3image
adpcm_ima_rad           |cdxl                    |gremlin_dpcm            |libopus                 |msvideo1                |prosumer                |tiff                    |wnv1
adpcm_ima_smjpeg        |cfhd                    |gsm                     |librsvg                 |mszh                    |psd                     |tmv                     |wrapped_avframe
adpcm_ima_ssi           |cinepak                 |gsm_ms                  |libspeex                |mts2                    |ptx                     |truehd                  |ws_snd1
adpcm_ima_wav           |clearvideo              |gsm_ms_at               |libvorbis               |mv30                    |qcelp                   |truemotion1             |xan_dpcm
adpcm_ima_ws            |cljr                    |h261                    |libvpx_vp8              |mvc1                    |qdm2                    |truemotion2             |xan_wc3
adpcm_ms                |cllc                    |h263                    |libvpx_vp9              |mvc2                    |qdm2_at                 |truemotion2rt           |xan_wc4
adpcm_mtaf              |comfortnoise            |h263_v4l2m2m            |libzvbi_teletext        |mvdv                    |qdmc                    |truespeech              |xbin
adpcm_psx               |cook                    |h263i                   |loco                    |mvha                    |qdmc_at                 |tscc                    |xbm
adpcm_sbpro_2           |cpia                    |h263p                   |lscr                    |mwsc                    |qdraw                   |tscc2                   |xface
adpcm_sbpro_3           |cscd                    |h264                    |m101                    |mxpeg                   |qpeg                    |tta                     |xl
adpcm_sbpro_4           |cyuv                    |h264_crystalhd          |mace3                   |nellymoser              |qtrle                   |twinvq                  |xma1
adpcm_swf               |dca                     |h264_cuvid              |mace6                   |notchlc                 |r10k                    |txd                     |xma2
adpcm_thp               |dds                     |h264_mediacodec         |magicyuv                |nuv                     |r210                    |ulti                    |xpm
adpcm_thp_le            |derf_dpcm               |h264_mmal               |mdec                    |on2avc                  |ra_144                  |utvideo                 |xsub
adpcm_vima              |dfa                     |h264_qsv                |metasound               |opus                    |ra_288                  |v210                    |xwd
adpcm_xa                |dirac                   |h264_rkmpp              |microdvd                |paf_audio               |ralf                    |v210x                   |y41p
adpcm_yamaha            |dnxhd                   |h264_v4l2m2m            |mimic                   |paf_video               |rasc                    |v308                    |ylc
adpcm_zork              |dolby_e                 |hap                     |mjpeg                   |pam                     |rawvideo                |v408                    |yop
agm                     |dpx                     |hca                     |mjpeg_cuvid             |pbm                     |realtext                |v410                    |yuv4
aic                     |dsd_lsbf                |hcom                    |mjpeg_qsv               |pcm_alaw                |rl2                     |vb                      |zero12v
alac                    |dsd_lsbf_planar         |hevc                    |mjpegb                  |pcm_alaw_at             |roq                     |vble                    |zerocodec
alac_at                 |dsd_msbf                |hevc_cuvid              |mlp                     |pcm_bluray              |roq_dpcm                |vc1                     |zlib
alias_pix               |dsd_msbf_planar         |hevc_mediacodec         |mmvideo                 |pcm_dvd                 |rpza                    |vc1_crystalhd           |zmbv
als                     |dsicinaudio             |hevc_qsv                |mobiclip                |pcm_f16le               |rscc                    |vc1_cuvid
amr_nb_at               |dsicinvideo             |hevc_rkmpp              |motionpixels            |pcm_f24le               |rv10                    |vc1_mmal
amrnb                   |dss_sp                  |hevc_v4l2m2m            |movtext                 |pcm_f32be               |rv20                    |vc1_qsv

### **encoders**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/15.png)

encoders||||||||
--|--
a64multi                |asv2                    |h264_amf                |libopenh264             |movtext                 |pcm_mulaw_at            |prores                  |utvideo
a64multi5               |avrp                    |h264_mf                 |libopenjpeg             |mp2                     |pcm_s16be               |prores_aw               |v210
aac                     |avui                    |h264_nvenc              |libopus                 |mp2fixed                |pcm_s16be_planar        |prores_ks               |v308
aac_at                  |ayuv                    |h264_omx                |librav1e                |mp3_mf                  |pcm_s16le               |qtrle                   |v408
aac_mf                  |bmp                     |h264_qsv                |libshine                |mpeg1video              |pcm_s16le_planar        |r10k                    |v410
ac3                     |cfhd                    |h264_v4l2m2m            |libspeex                |mpeg2_qsv               |pcm_s24be               |r210                    |vc2
ac3_fixed               |cinepak                 |h264_vaapi              |libsvtav1               |mpeg2_vaapi             |pcm_s24daud             |ra_144                  |vorbis
ac3_mf                  |cljr                    |h264_videotoolbox       |libtheora               |mpeg2video              |pcm_s24le               |rawvideo                |vp8_v4l2m2m
adpcm_adx               |comfortnoise            |hap                     |libtwolame              |mpeg4                   |pcm_s24le_planar        |roq                     |vp8_vaapi
adpcm_argo              |dca                     |hevc_amf                |libvo_amrwbenc          |mpeg4_omx               |pcm_s32be               |roq_dpcm                |vp9_qsv
adpcm_g722              |dnxhd                   |hevc_mf                 |libvorbis               |mpeg4_v4l2m2m           |pcm_s32le               |rpza                    |vp9_vaapi
adpcm_g726              |dpx                     |hevc_nvenc              |libvpx_vp8              |msmpeg4v2               |pcm_s32le_planar        |rv10                    |wavpack
adpcm_g726le            |dvbsub                  |hevc_qsv                |libvpx_vp9              |msmpeg4v3               |pcm_s64be               |rv20                    |webvtt
adpcm_ima_apm           |dvdsub                  |hevc_v4l2m2m            |libwavpack              |msvideo1                |pcm_s64le               |s302m                   |wmav1
adpcm_ima_qt            |dvvideo                 |hevc_vaapi              |libwebp                 |nellymoser              |pcm_s8                  |sbc                     |wmav2
adpcm_ima_ssi           |eac3                    |hevc_videotoolbox       |libwebp_anim            |nvenc                   |pcm_s8_planar           |sgi                     |wmv1
adpcm_ima_wav           |ffv1                    |huffyuv                 |libx262                 |nvenc_h264              |pcm_u16be               |snow                    |wmv2
adpcm_ms                |ffvhuff                 |ilbc_at                 |libx264                 |nvenc_hevc              |pcm_u16le               |sonic                   |wrapped_avframe
adpcm_swf               |fits                    |jpeg2000                |libx264rgb              |opus                    |pcm_u24be               |sonic_ls                |xbm
adpcm_yamaha            |flac                    |jpegls                  |libx265                 |pam                     |pcm_u24le               |srt                     |xface
alac                    |flashsv                 |libaom_av1              |libxavs                 |pbm                     |pcm_u32be               |ssa                     |xsub
alac_at                 |flashsv2                |libcodec2               |libxavs2                |pcm_alaw                |pcm_u32le               |subrip                  |xwd
alias_pix               |flv                     |libfdk_aac              |libxvid                 |pcm_alaw_at             |pcm_u8                  |sunrast                 |y41p
amv                     |g723_1                  |libgsm                  |ljpeg                   |pcm_dvd                 |pcm_vidc                |svq1                    |yuv4
apng                    |gif                     |libgsm_ms               |magicyuv                |pcm_f32be               |pcx                     |targa                   |zlib
aptx                    |h261                    |libilbc                 |mjpeg                   |pcm_f32le               |pgm                     |text                    |zmbv
aptx_hd                 |h263                    |libkvazaar              |mjpeg_qsv               |pcm_f64be               |pgmyuv                  |tiff
ass                     |h263_v4l2m2m            |libmp3lame              |mjpeg_vaapi             |pcm_f64le               |png                     |truehd
asv1                    |h263p                   |libopencore_amrnb       |mlp                     |pcm_mulaw               |ppm                     |tta

### **hardware accelerators**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/16.png)

hardware accelerators||||||||
--|--
h263_vaapi              |h264_vdpau              |hevc_vdpau              |mpeg1_xvmc              |mpeg2_videotoolbox      |vc1_d3d11va2            |vp9_d3d11va             |wmv3_d3d11va2
h263_videotoolbox       |h264_videotoolbox       |hevc_videotoolbox       |mpeg2_d3d11va           |mpeg2_xvmc              |vc1_dxva2               |vp9_d3d11va2            |wmv3_dxva2
h264_d3d11va            |hevc_d3d11va            |mjpeg_nvdec             |mpeg2_d3d11va2          |mpeg4_nvdec             |vc1_nvdec               |vp9_dxva2               |wmv3_nvdec
h264_d3d11va2           |hevc_d3d11va2           |mjpeg_vaapi             |mpeg2_dxva2             |mpeg4_vaapi             |vc1_vaapi               |vp9_nvdec               |wmv3_vaapi
h264_dxva2              |hevc_dxva2              |mpeg1_nvdec             |mpeg2_nvdec             |mpeg4_vdpau             |vc1_vdpau               |vp9_vaapi               |wmv3_vdpau
h264_nvdec              |hevc_nvdec              |mpeg1_vdpau             |mpeg2_vaapi             |mpeg4_videotoolbox      |vp8_nvdec               |vp9_vdpau
h264_vaapi              |hevc_vaapi              |mpeg1_videotoolbox      |mpeg2_vdpau             |vc1_d3d11va             |vp8_vaapi               |wmv3_d3d11va

### **demuxers**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/17.png)

demuxers||||||||
--|--
aa                      |bfstm                   |flac                    |image_jpeg_pipe         |lxf                     |obu                     |rtp                     |txd
aac                     |bink                    |flic                    |image_jpegls_pipe       |m4v                     |ogg                     |rtsp                    |ty
aax                     |bintext                 |flv                     |image_pam_pipe          |matroska                |oma                     |s337m                   |v210
ac3                     |bit                     |fourxm                  |image_pbm_pipe          |mca                     |paf                     |sami                    |v210x
acm                     |bmv                     |frm                     |image_pcx_pipe          |mcc                     |pcm_alaw                |sap                     |vag
act                     |boa                     |fsb                     |image_pgm_pipe          |mgsts                   |pcm_f32be               |sbc                     |vapoursynth
adf                     |brstm                   |fwse                    |image_pgmyuv_pipe       |microdvd                |pcm_f32le               |sbg                     |vc1
adp                     |c93                     |g722                    |image_pgx_pipe          |mjpeg                   |pcm_f64be               |scc                     |vc1t
ads                     |caf                     |g723_1                  |image_photocd_pipe      |mjpeg_2000              |pcm_f64le               |sdp                     |vividas
adx                     |cavsvideo               |g726                    |image_pictor_pipe       |mlp                     |pcm_mulaw               |sdr2                    |vivo
aea                     |cdg                     |g726le                  |image_png_pipe          |mlv                     |pcm_s16be               |sds                     |vmd
afc                     |cdxl                    |g729                    |image_ppm_pipe          |mm                      |pcm_s16le               |sdx                     |vobsub
aiff                    |cine                    |gdv                     |image_psd_pipe          |mmf                     |pcm_s24be               |segafilm                |voc
aix                     |codec2                  |genh                    |image_qdraw_pipe        |mods                    |pcm_s24le               |ser                     |vpk
alp                     |codec2raw               |gif                     |image_sgi_pipe          |moflex                  |pcm_s32be               |shorten                 |vplayer
amr                     |concat                  |gsm                     |image_sunrast_pipe      |mov                     |pcm_s32le               |siff                    |vqf
amrnb                   |dash                    |gxf                     |image_svg_pipe          |mp3                     |pcm_s8                  |sln                     |w64
amrwb                   |data                    |h261                    |image_tiff_pipe         |mpc                     |pcm_u16be               |smacker                 |wav
anm                     |daud                    |h263                    |image_webp_pipe         |mpc8                    |pcm_u16le               |smjpeg                  |wc3
apc                     |dcstr                   |h264                    |image_xpm_pipe          |mpegps                  |pcm_u24be               |smush                   |webm_dash_manifest
ape                     |derf                    |hca                     |image_xwd_pipe          |mpegts                  |pcm_u24le               |sol                     |webvtt
apm                     |dfa                     |hcom                    |ingenient               |mpegtsraw               |pcm_u32be               |sox                     |wsaud
apng                    |dhav                    |hevc                    |ipmovie                 |mpegvideo               |pcm_u32le               |spdif                   |wsd
aptx                    |dirac                   |hls                     |ircam                   |mpjpeg                  |pcm_u8                  |srt                     |wsvqa
aptx_hd                 |dnxhd                   |hnm                     |iss                     |mpl2                    |pcm_vidc                |stl                     |wtv
aqtitle                 |dsf                     |ico                     |iv8                     |mpsub                   |pjs                     |str                     |wv
argo_asf                |dsicin                  |idcin                   |ivf                     |msf                     |pmp                     |subviewer               |wve
argo_brp                |dss                     |idf                     |ivr                     |msnwc_tcp               |pp_bnk                  |subviewer1              |xa
asf                     |dts                     |iff                     |jacosub                 |mtaf                    |pva                     |sup                     |xbin
asf_o                   |dtshd                   |ifv                     |jv                      |mtv                     |pvf                     |svag                    |xmv
ass                     |dv                      |ilbc                    |kux                     |musx                    |qcp                     |svs                     |xvag
ast                     |dvbsub                  |image2                  |kvag                    |mv                      |r3d                     |swf                     |xwma
au                      |dvbtxt                  |image2_alias_pix        |libgme                  |mvi                     |rawvideo                |tak                     |yop
av1                     |dxa                     |image2_brender_pix      |libmodplug              |mxf                     |realtext                |tedcaptions             |yuv4mpegpipe
avi                     |ea                      |image2pipe              |libopenmpt              |mxg                     |redspark                |thp
avisynth                |ea_cdata                |image_bmp_pipe          |live_flv                |nc                      |rl2                     |threedostr
avr                     |eac3                    |image_dds_pipe          |lmlm4                   |nistsphere              |rm                      |tiertexseq
avs                     |epaf                    |image_dpx_pipe          |loas                    |nsp                     |roq                     |tmv
avs2                    |ffmetadata              |image_exr_pipe          |lrc                     |nsv                     |rpl                     |truehd
bethsoftvid             |filmstrip               |image_gif_pipe          |luodat                  |nut                     |rsd                     |tta
bfi                     |fits                    |image_j2k_pipe          |lvf                     |nuv                     |rso                     |tty

### **muxers**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/18.png)

muxers||||||||
--|--
a64                     |cavsvideo               |framecrc                |ipod                    |mpeg1system             |pcm_f32le               |rm                      |swf
ac3                     |chromaprint             |framehash               |ircam                   |mpeg1vcd                |pcm_f64be               |roq                     |tee
adts                    |codec2                  |framemd5                |ismv                    |mpeg1video              |pcm_f64le               |rso                     |tg2
adx                     |codec2raw               |g722                    |ivf                     |mpeg2dvd                |pcm_mulaw               |rtp                     |tgp
aiff                    |crc                     |g723_1                  |jacosub                 |mpeg2svcd               |pcm_s16be               |rtp_mpegts              |truehd
amr                     |dash                    |g726                    |kvag                    |mpeg2video              |pcm_s16le               |rtsp                    |tta
apm                     |data                    |g726le                  |latm                    |mpeg2vob                |pcm_s24be               |sap                     |uncodedframecrc
apng                    |daud                    |gif                     |lrc                     |mpegts                  |pcm_s24le               |sbc                     |vc1
aptx                    |dirac                   |gsm                     |m4v                     |mpjpeg                  |pcm_s32be               |scc                     |vc1t
aptx_hd                 |dnxhd                   |gxf                     |matroska                |mxf                     |pcm_s32le               |segafilm                |voc
argo_asf                |dts                     |h261                    |matroska_audio          |mxf_d10                 |pcm_s8                  |segment                 |w64
asf                     |dv                      |h263                    |md5                     |mxf_opatom              |pcm_u16be               |singlejpeg              |wav
asf_stream              |eac3                    |h264                    |microdvd                |null                    |pcm_u16le               |smjpeg                  |webm
ass                     |f4v                     |hash                    |mjpeg                   |nut                     |pcm_u24be               |smoothstreaming         |webm_chunk
ast                     |ffmetadata              |hds                     |mkvtimestamp_v2         |oga                     |pcm_u24le               |sox                     |webm_dash_manifest
au                      |fifo                    |hevc                    |mlp                     |ogg                     |pcm_u32be               |spdif                   |webp
avi                     |fifo_test               |hls                     |mmf                     |ogv                     |pcm_u32le               |spx                     |webvtt
avm2                    |filmstrip               |ico                     |mov                     |oma                     |pcm_u8                  |srt                     |wtv
avs2                    |fits                    |ilbc                    |mp2                     |opus                    |pcm_vidc                |stream_segment          |wv
bit                     |flac                    |image2                  |mp3                     |pcm_alaw                |psp                     |streamhash              |yuv4mpegpipe
caf                     |flv                     |image2pipe              |mp4                     |pcm_f32be               |rawvideo                |sup

### **parsers**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/19.png)

parsers||||||||
--|--
aac                     |bmp                     |dpx                     |g723_1                  |h264                    |mpegaudio               |rv40                    |vp3
aac_latm                |cavsvideo               |dvaudio                 |g729                    |hevc                    |mpegvideo               |sbc                     |vp8
ac3                     |cook                    |dvbsub                  |gif                     |jpeg2000                |opus                    |sipr                    |vp9
adx                     |dca                     |dvd_nav                 |gsm                     |mjpeg                   |png                     |tak                     |webp
av1                     |dirac                   |dvdsub                  |h261                    |mlp                     |pnm                     |vc1                     |xma
avs2                    |dnxhd                   |flac                    |h263                    |mpeg4video              |rv30                    |vorbis

### **protocols**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/20.png)

protocols||||||||
--|--
async                   |ffrtmpcrypt             |http                    |librtmpe                |libssh                  |prompeg                 |rtmpts                  |tee
bluray                  |ffrtmphttp              |httpproxy               |librtmps                |libzmq                  |rtmp                    |rtp                     |tls
cache                   |file                    |https                   |librtmpt                |md5                     |rtmpe                   |sctp                    |udp
concat                  |ftp                     |icecast                 |librtmpte               |mmsh                    |rtmps                   |srtp                    |udplite
crypto                  |gopher                  |libamqp                 |libsmbclient            |mmst                    |rtmpt                   |subfile                 |unix
data                    |hls                     |librtmp                 |libsrt                  |pipe                    |rtmpte                  |tcp

### **bitstream filters**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/21.png)

bitstream filters||||||||
--|--
aac_adtstoasc           |dca_core                |h264_metadata           |hevc_mp4toannexb        |mp3_header_decompress   |opus_metadata           |trace_headers           |vp9_superframe_split
av1_frame_merge         |dump_extradata          |h264_mp4toannexb        |imx_dump_header         |mpeg2_metadata          |pcm_rechunk             |truehd_core
av1_frame_split         |eac3_core               |h264_redundant_pps      |mjpeg2jpeg              |mpeg4_unpack_bframes    |prores_metadata         |vp9_metadata
av1_metadata            |extract_extradata       |hapqa_extract           |mjpega_dump_header      |noise                   |remove_extradata        |vp9_raw_reorder
chomp                   |filter_units            |hevc_metadata           |mov2textsub             |null                    |text2movsub             |vp9_superframe

### **input devices**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/22.png)

input devices|||||||
--|--
alsa                    |bktr                    |fbdev                   |jack                    |libcdio                 |oss                     |v4l2
android_camera          |decklink                |gdigrab                 |kmsgrab                 |libdc1394               |pulse                   |vfwcap
avfoundation            |dshow                   |iec61883                |lavfi                   |openal                  |sndio                   |xcbgrab

### **output devices**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/23.png)

output devices||||||
--|--
alsa                    |caca                    |fbdev                   |oss                     |sdl2                    |v4l2
audiotoolbox            |decklink                |opengl                  |pulse                   |sndio                   |xv

### **filters**
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/24.png)

filters||||||||
--|--
abench                  |aresample               |colorhold               |elbg                    |idet                    |oscilloscope            |select                  |testsrc2
abitscope               |areverse                |colorkey                |entropy                 |il                      |overlay                 |selectivecolor          |thistogram
acompressor             |arnndn                  |colorkey_opencl         |eq                      |inflate                 |overlay_cuda            |sendcmd                 |threshold
acontrast               |aselect                 |colorlevels             |equalizer               |interlace               |overlay_opencl          |separatefields          |thumbnail
acopy                   |asendcmd                |colormatrix             |erosion                 |interleave              |overlay_qsv             |setdar                  |thumbnail_cuda
acrossfade              |asetnsamples            |colorspace              |erosion_opencl          |join                    |overlay_vulkan          |setfield                |tile
acrossover              |asetpts                 |compand                 |extractplanes           |kerndeint               |owdenoise               |setparams               |tinterlace
acrusher                |asetrate                |compensationdelay       |extrastereo             |ladspa                  |pad                     |setpts                  |tlut2
acue                    |asettb                  |concat                  |fade                    |lagfun                  |pad_opencl              |setrange                |tmedian
addroi                  |ashowinfo               |convolution             |fftdnoiz                |lenscorrection          |pal100bars              |setsar                  |tmix
adeclick                |asidedata               |convolution_opencl      |fftfilt                 |lensfun                 |pal75bars               |settb                   |tonemap
adeclip                 |asoftclip               |convolve                |field                   |libvmaf                 |palettegen              |sharpness_vaapi         |tonemap_opencl
adelay                  |asplit                  |copy                    |fieldhint               |life                    |paletteuse              |showcqt                 |tonemap_vaapi
aderivative             |asr                     |coreimage               |fieldmatch              |limiter                 |pan                     |showfreqs               |tpad
adrawgraph              |ass                     |coreimagesrc            |fieldorder              |loop                    |perms                   |showinfo                |transpose
aecho                   |astats                  |cover_rect              |fifo                    |loudnorm                |perspective             |showpalette             |transpose_npp
aemphasis               |astreamselect           |crop                    |fillborders             |lowpass                 |phase                   |showspatial             |transpose_opencl
aeval                   |asubboost               |cropdetect              |find_rect               |lowshelf                |photosensitivity        |showspectrum            |transpose_vaapi
aevalsrc                |atadenoise              |crossfeed               |firequalizer            |lumakey                 |pixdesctest             |showspectrumpic         |treble
afade                   |atempo                  |crystalizer             |flanger                 |lut                     |pixscope                |showvolume              |tremolo
afftdn                  |atrim                   |cue                     |flite                   |lut1d                   |pp                      |showwaves               |trim
afftfilt                |avectorscope            |curves                  |floodfill               |lut2                    |pp7                     |showwavespic            |unpremultiply
afifo                   |avgblur                 |datascope               |format                  |lut3d                   |premultiply             |shuffleframes           |unsharp
afir                    |avgblur_opencl          |dblur                   |fps                     |lutrgb                  |prewitt                 |shuffleplanes           |unsharp_opencl
afirsrc                 |avgblur_vulkan          |dcshift                 |framepack               |lutyuv                  |prewitt_opencl          |sidechaincompress       |untile
aformat                 |axcorrelate             |dctdnoiz                |framerate               |lv2                     |procamp_vaapi           |sidechaingate           |uspp
agate                   |azmq                    |deband                  |framestep               |mandelbrot              |program_opencl          |sidedata                |v360
agraphmonitor           |bandpass                |deblock                 |freezedetect            |maskedclamp             |pseudocolor             |sierpinski              |vaguedenoiser
ahistogram              |bandreject              |decimate                |freezeframes            |maskedmax               |psnr                    |signalstats             |vectorscope
aiir                    |bass                    |deconvolve              |frei0r                  |maskedmerge             |pullup                  |signature               |vflip
aintegral               |bbox                    |dedot                   |frei0r_src              |maskedmin               |qp                      |silencedetect           |vfrdet
ainterleave             |bench                   |deesser                 |fspp                    |maskedthreshold         |random                  |silenceremove           |vibrance
alimiter                |bilateral               |deflate                 |gblur                   |maskfun                 |readeia608              |sinc                    |vibrato
allpass                 |biquad                  |deflicker               |geq                     |mcdeint                 |readvitc                |sine                    |vidstabdetect
allrgb                  |bitplanenoise           |deinterlace_qsv         |gradfun                 |mcompand                |realtime                |smartblur               |vidstabtransform
allyuv                  |blackdetect             |deinterlace_vaapi       |gradients               |median                  |remap                   |smptebars               |vignette
aloop                   |blackframe              |dejudder                |graphmonitor            |mergeplanes             |removegrain             |smptehdbars             |vmafmotion
alphaextract            |blend                   |delogo                  |greyedge                |mestimate               |removelogo              |sobel                   |volume
alphamerge              |bm3d                    |denoise_vaapi           |haas                    |metadata                |repeatfields            |sobel_opencl            |volumedetect
amerge                  |boxblur                 |derain                  |haldclut                |midequalizer            |replaygain              |sofalizer               |vpp_qsv
ametadata               |boxblur_opencl          |deshake                 |haldclutsrc             |minterpolate            |resample                |spectrumsynth           |vstack
amix                    |bs2b                    |deshake_opencl          |hdcd                    |mix                     |reverse                 |split                   |w3fdif
amovie                  |bwdif                   |despill                 |headphone               |movie                   |rgbashift               |spp                     |waveform
amplify                 |cas                     |detelecine              |hflip                   |mpdecimate              |rgbtestsrc              |sr                      |weave
amultiply               |cellauto                |dilation                |highpass                |mptestsrc               |roberts                 |ssim                    |xbr
anequalizer             |channelmap              |dilation_opencl         |highshelf               |negate                  |roberts_opencl          |stereo3d                |xfade
anlmdn                  |channelsplit            |displace                |hilbert                 |nlmeans                 |rotate                  |stereotools             |xfade_opencl
anlms                   |chorus                  |dnn_processing          |histeq                  |nlmeans_opencl          |rubberband              |stereowiden             |xmedian
anoisesrc               |chromaber_vulkan        |doubleweave             |histogram               |nnedi                   |sab                     |streamselect            |xstack
anull                   |chromahold              |drawbox                 |hqdn3d                  |noformat                |scale                   |subtitles               |yadif
anullsink               |chromakey               |drawgraph               |hqx                     |noise                   |scale2ref               |super2xsai              |yadif_cuda
anullsrc                |chromanr                |drawgrid                |hstack                  |normalize               |scale_cuda              |superequalizer          |yaepblur
apad                    |chromashift             |drawtext                |hue                     |null                    |scale_npp               |surround                |yuvtestsrc
aperms                  |ciescope                |drmeter                 |hwdownload              |nullsink                |scale_qsv               |swaprect                |zmq
aphasemeter             |codecview               |dynaudnorm              |hwmap                   |nullsrc                 |scale_vaapi             |swapuv                  |zoompan
aphaser                 |color                   |earwax                  |hwupload                |ocr                     |scale_vulkan            |tblend                  |zscale
apulsator               |colorbalance            |ebur128                 |hwupload_cuda           |ocv                     |scdet                   |telecine
arealtime               |colorchannelmixer       |edgedetect              |hysteresis              |openclsrc               |scroll                  |testsrc

## **部分命令说明**
Standard options|描述
--|--
--logfile=FILE           |记录测试并输出到文件 FILE
--disable-logging        |
--fatal-warnings         |
--prefix=PREFIX          |安装程序到指定目录
--bindir=DIR             |
--datadir=DIR            |
--docdir=DIR             |
--libdir=DIR             |
--shlibdir=DIR           |
--incdir=DIR             |
--mandir=DIR             |
--pkgconfigdir=DIR       |
--enable-rpath           |
--install-name-dir=DIR   |

Licensing options|描述
--|--
--enable-gpl             |只要这种修改文本在整体上或者其某个部分来源于遵循 GPL 的程序，该修改文本的整体就必须按照 GPL 流通，不仅该修改文本的源码必须向社会公开，而且对于这种修改文本的流通不准许附加修改者自己作出的限制。
--enable-version3        |
--enable-nonfree         |

Configuration options|描述
--|--
--disable-static         |不生成静态库（no）
--enable-shared          |生成动态库（no）
--enable-small           |针对 size 优化，而针对不是速度优化
--disable-runtime-cpudetect|禁止在运行时检测 CPU 适配性
--enable-gray            |启用完全的 grayscale 支持
--disable-swscale-alpha  |禁止 swscale 的 alpha 通道
--disable-all            |禁止生成文件
--disable-autodetect     |禁止自动检测外部的库（no）

Program options|描述
--|--
--disable-programs       |不生成命令行程序
--disable-ffmpeg         |不生成 ffmpeg
--disable-ffplay         |不生成 ffplay
--disable-ffprobe        |不生成 ffprobe

Documentation options|描述
--|--
--disable-doc            |不生成文档
--disable-htmlpages      |不生成 HTML 文档
--disable-manpages       |不生成 man 文档
--disable-podpages       |不生成 POD 文档
--disable-txtpages       |不生成 text 文档

Component options|描述
--|--
--disable-avdevice       |不生成 libavdevice
--disable-avcodec        |不生成 libavcodec
--disable-avformat       |不生成 libavformat
--disable-swresample     |不生成 libswresample
--disable-swscale        |不生成 libswscale
--disable-postproc       |不生成 libpostproc
--disable-avfilter       |不生成 libavfilter
--enable-avresample      |生成 libavresample（已废弃）（no）
--disable-pthreads       |禁止 pthreads
--disable-w32threads     |禁止 win32 threads
--disable-os2threads     |禁止 OS/2 threads
--disable-network        |禁止网络支持
--disable-dct            |禁止 DCT
--disable-dwt            |禁止 DWT
--disable-error-resilience|禁止错误恢复
--disable-lsp            |禁止 LSP
--disable-lzo            |禁止 LZO 解码
--disable-mdct           |禁止 MDCT
--disable-rdft           |禁止 RDFT
--disable-fft            |禁止 FFT
--disable-faan           |禁止浮点 AAN（I）DCT
--disable-pixelutils     |禁止 libavutil 中的 pixel util

Individual component options|描述
--|--
--disable-everything     |禁止以下所有组件
--disable-encoder=NAME   |禁止编码器 NAME
--enable-encoder=NAME    |允许解码器 NAME
--disable-encoders       |禁止所有编码器
--disable-decoder=NAME   |禁止解码器 NAME
--enable-decoder=NAME    |允许解码器 NAME
--disable-decoders       |禁止所有解码器
--disable-hwaccel=NAME   |禁止硬件加速 hwaccel
--enable-hwaccel=NAME    |允许硬件加速 hwaccel
--disable-hwaccels       |禁止所有硬件加速
--disable-muxer=NAME     |禁止封装 NAME
--enable-muxer=NAME      |允许封装 NAME
--disable-muxers         |禁止所有封装
--disable-demuxer=NAME   |禁止解封装 NAME
--enable-demuxer=NAME    |允许解封装 NAME
--disable-demuxers       |禁止所有解封装
--enable-parser=NAME     |允许解析 NAME
--disable-parser=NAME    |禁止解析 NAME
--disable-parsers        |禁止所有解析 NAME
--enable-bsf=NAME        |允许比特流过滤器 NAME
--disable-bsf=NAME       |禁止比特流过滤器 NAME
--disable-bsfs           |禁止所有比特流过滤器 NAME
--enable-protocol=NAME   |
--disable-protocol=NAME  |
--disable-protocols      |
--enable-indev=NAME      |
--disable-indev=NAME     |
--disable-indevs         |
--enable-outdev=NAME     |
--disable-outdev=NAME    |
--disable-outdevs        |
--disable-devices        |
--enable-filter=NAME     |允许过滤器 NAME
--disable-filter=NAME    |禁止过滤器 NAME
--disable-filters        |禁止所有过滤器 NAME

External library support|描述
--|--
--disable-alsa           |
--disable-appkit         |禁用 Apple AppKit 框架
--disable-avfoundation   |禁用 Apple AVFoundation 框架
--enable-avisynth        |
--disable-bzlib          |
--disable-coreimage      |禁用 Apple CoreImage 框架
--enable-chromaprint     |
--enable-frei0r          |
--enable-gcrypt          |
--enable-gmp             |
--enable-gnutls          |
--disable-iconv          |
--enable-jni             |允许 JNI 支持（no）
--enable-ladspa          |
--enable-libaom          |
--enable-libaribb24      |
--enable-libass          |
--enable-libbluray       |
--enable-libbs2b         |
--enable-libcaca         |
--enable-libcelt         |
--enable-libcdio         |
--enable-libcodec2       |
--enable-libdav1d        |
--enable-libdavs2        |
--enable-libdc1394       |
--enable-libfdk-aac      |
--enable-libflite        |
--enable-libfontconfig   |
--enable-libfreetype     |
--enable-libfribidi      |
--enable-libglslang      |
--enable-libgme          |
--enable-libgsm          |
--enable-libiec61883     |
--enable-libilbc         |
--enable-libjack         |
--enable-libklvanc       |
--enable-libkvazaar      |
--enable-liblensfun      |
--enable-libmodplug      |
--enable-libmp3lame      |
--enable-libopencore-amrnb|
--enable-libopencore-amrwb|
--enable-libopencv       |
--enable-libopenh264     |
--enable-libopenjpeg     |
--enable-libopenmpt      |
--enable-libopenvino     |
--enable-libopus         |
--enable-libpulse        |
--enable-librabbitmq     |
--enable-librav1e        |
--enable-librsvg         |
--enable-librubberband   |
--enable-librtmp         |
--enable-libshine        |
--enable-libsmbclient    |
--enable-libsnappy       |
--enable-libsoxr         |
--enable-libspeex        |
--enable-libsrt          |
--enable-libssh          |
--enable-libsvtav1       |
--enable-libtensorflow   |
--enable-libtesseract    |
--enable-libtheora       |
--enable-libtls          |
--enable-libtwolame      |
--enable-libv4l2         |
--enable-libvidstab      |
--enable-libvmaf         |
--enable-libvo-amrwbenc  |
--enable-libvorbis       |
--enable-libvpx          |
--enable-libwavpack      |
--enable-libwebp         |
--enable-libx264         |
--enable-libx265         |
--enable-libxavs         |
--enable-libxavs2        |
--enable-libxcb          |
--enable-libxcb-shm      |
--enable-libxcb-xfixes   |
--enable-libxcb-shape    |
--enable-libxvid         |
--enable-libxml2         |
--enable-libzimg         |
--enable-libzmq          |
--enable-libzvbi         |
--enable-lv2             |
--disable-lzma           |
--enable-decklink        |
--enable-mbedtls         |
--enable-mediacodec      |支持 Android MediaCodec
--enable-mediafoundation |
--enable-libmysofa       |
--enable-openal          |
--enable-opencl          |
--enable-opengl          |
--enable-openssl         |
--enable-pocketsphinx    |
--disable-sndio          |
--disable-schannel       |
--disable-sdl2           |
--disable-securetransport|
--enable-vapoursynth     |
--enable-vulkan          |
--disable-xlib           |
--disable-zlib           |

External library support|描述
--|--
--disable-amf            |
--disable-audiotoolbox   |禁用 Apple AudioToolbox 代码
--enable-cuda-nvcc       |
--disable-cuda-llvm      |
--disable-cuvid          |
--disable-d3d11va        |
--disable-dxva2          |
--disable-ffnvcodec      |
--enable-libdrm          |
--enable-libmfx          |
--enable-libnpp          |
--enable-mmal            |
--disable-nvdec          |
--disable-nvenc          |
--enable-omx             |
--enable-omx-rpi         |
--enable-rkmpp           |
--disable-v4l2-m2m       |
--disable-vaapi          |
--disable-vdpau          |
--disable-videotoolbox   |

Toolchain options|描述
--|--
--arch=ARCH              |架构
--cpu=CPU                |最小 CPU
--cross-prefix=PREFIX    |编译工具前缀 PREFIX
--progs-suffix=SUFFIX    |程序名后缀 SUFFIX
--enable-cross-compile   |使用交叉编译器
--sysroot=PATH           |sysroot PATH
--sysinclude=PATH        |
--target-os=OS           |target-os OS
--target-exec=CMD        |
--target-path=DIR        |
--target-samples=DIR     |
--tempprefix=PATH        |
--toolchain=NAME         |toolchain NAME
--nm=NM                  |nm NM
--ar=AR                  |ar AR
--as=AS                  |as AS
--ln_s=LN_S              |
--strip=STRIP            |strip STRIP
--windres=WINDRES        |
--x86asmexe=EXE          |
--cc=CC                  |C 编译器 CC
--cxx=CXX                |C++ 编译器 CXX
--objcc=OCC              |
--dep-cc=DEPCC           |
--nvcc=NVCC              |
--ld=LD                  |ld LD
--pkg-config=PKGCONFIG   |
--pkg-config-flags=FLAGS |
--ranlib=RANLIB          |ranlib RANLIB
--doxygen=DOXYGEN        |
--host-cc=HOSTCC         |
--host-cflags=HCFLAGS    |
--host-cppflags=HCPPFLAGS|
--host-ld=HOSTLD         |
--host-ldflags=HLDFLAGS  |
--host-extralibs=HLIBS   |
--host-os=OS             |
--extra-cflags=ECFLAGS   |CFLAGS
--extra-cxxflags=ECFLAGS |
--extra-objcflags=FLAGS  |
--extra-ldflags=ELDFLAGS |
--extra-ldexeflags=ELDFLAGS|
--extra-ldsoflags=ELDFLAGS|
--extra-libs=ELIBS       |
--extra-version=STRING   |
--optflags=OPTFLAGS      |
--nvccflags=NVCCFLAGS    |
--build-suffix=SUFFIX    |
--enable-pic             |允许 PIC
--enable-thumb           |
--enable-lto             |
--env="ENV=override"     |

Advanced options (experts only)|描述
--|--
--malloc-prefix=PREFIX   |
--custom-allocator=NAME  |
--disable-symver         |
--enable-hardcoded-tables|
--disable-safe-bitstream-reader|
--sws-max-filter-size=N  |

Optimization options (experts only)|描述
--|--
--disable-asm            |
--disable-altivec        |
--disable-vsx            |
--disable-power8         |
--disable-amd3dnow       |
--disable-amd3dnowext    |
--disable-mmx            |
--disable-mmxext         |
--disable-sse            |
--disable-sse2           |
--disable-sse3           |
--disable-ssse3          |
--disable-sse4           |
--disable-sse42          |
--disable-avx            |
--disable-xop            |
--disable-fma3           |
--disable-fma4           |
--disable-avx2           |
--disable-avx512         |
--disable-aesni          |
--disable-armv5te        |
--disable-armv6          |
--disable-armv6t2        |
--disable-vfp            |
--disable-neon           |用 ARM 扩展的 neon 指令集可以很大的优化 FFmpeg 的解码功能
--disable-inline-asm     |
--disable-x86asm         |
--disable-mipsdsp        |
--disable-mipsdspr2      |
--disable-msa            |
--disable-msa2           |
--disable-mipsfpu        |
--disable-mmi            |
--disable-fast-unaligned |

Developer options (useful when working on FFmpeg itself)|描述
--|--
--disable-debug          |
--enable-debug=LEVEL     |设置调试级别 LEVEL
--disable-optimizations  |
--enable-extra-warnings  |启用更多编译器警告
--disable-stripping      |
--assert-level=level     |
--enable-memory-poisoning|
--valgrind=VALGRIND      |
--enable-ftrapv          |
--samples=PATH           |
--enable-neon-clobber-test|
--enable-xmm-clobber-test|
--enable-random          |
--disable-random         |
--enable-random=LIST     |
--disable-random=LIST    |
--random-seed=VALUE      |
--disable-valgrind-backtrace|
--enable-ossfuzz         |
--libfuzzer=PATH         |
--ignore-tests=TESTS     |
--enable-linux-perf      |
--disable-large-tests    |

## **编写脚本文件**
1、创建文件（创建成 .txt 文件可以直接打开，但是 .sh 有文字颜色）
```shell
gedit build.sh
```

2、脚本文件：
```shell
#!/bin/sh

NDK=/home/weichao/android-ndk-r21d
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64
SYSROOT=$TOOLCHAIN/sysroot
API=21

function my_build
{
echo "Compiling FFmpeg for $CPU..."

./configure \
\
--logfile=$LOGFILE \
--prefix=$PREFIX \
\
--enable-gpl \
\
--disable-static \
--enable-shared \
--disable-runtime-cpudetect \
\
--disable-programs \
\
--disable-doc \
\
--disable-avdevice \
--disable-postproc \
--disable-w32threads \
--disable-os2threads \
\
--enable-decoder=h264_mediacodec \
\
--enable-jni \
--enable-mediacodec \
\
--arch=$ARCH \
--cpu=$CPU  \
--cross-prefix=$CROSS_PREFIX \
--enable-cross-compile \
--sysroot=$SYSROOT \
--target-os=android \
--cc=$CC \
--cxx=$CXX \
--extra-cflags="$EXTRA_CFLAGS" \
--enable-pic \
\
--enable-neon \

make clean
make -j8
make install

echo "The Compilation of FFmpeg for $CPU is completed."
}

#armeabi-v7a
CPU=armv7-a
ARCH=arm
ANDROID_ARCH_ABI=armeabi
LOGFILE=`pwd`/Android/log_armeabi-v7a.txt
PREFIX=`pwd`/Android/armeabi-v7a
CC=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang
CXX=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang++
EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -march=armv7-a -mthumb -Wformat -Werror=format-security   -Oz -DNDEBUG  -fPIC --gcc-toolchain=$TOOLCHAIN"
CROSS_PREFIX=$TOOLCHAIN/bin/arm-linux-androideabi-
my_build

#arm64-v8a
CPU=armv8-a
ARCH=arm64
ANDROID_ARCH_ABI=aarch64
LOGFILE=`pwd`/Android/log_arm64-v8a.txt
PREFIX=`pwd`/Android/arm64-v8a
CC=$TOOLCHAIN/bin/aarch64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/aarch64-linux-android$API-clang++
EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=aarch64-none-linux-android21 --gcc-toolchain=$TOOLCHAIN"
CROSS_PREFIX=$TOOLCHAIN/bin/aarch64-linux-android-
my_build

#x86_64
CPU=x86_64
ARCH=x86_64
LOGFILE=`pwd`/Android/log_x86_64.txt
PREFIX=`pwd`/Android/x86_64
CC=$TOOLCHAIN/bin/x86_64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/x86_64-linux-android$API-clang++
EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=x86_64-none-linux-android21 --gcc-toolchain=$TOOLCHAIN"
CROSS_PREFIX=$TOOLCHAIN/bin/x86_64-linux-android-
my_build
```

3、x86 多一个配置参数，单独一个文件：
```shell
#!/bin/sh

NDK=/home/weichao/android-ndk-r21d
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64
SYSROOT=$TOOLCHAIN/sysroot
API=21

function my_build
{
echo "Compiling FFmpeg for $CPU..."

./configure \
\
--logfile=$LOGFILE \
--prefix=$PREFIX \
\
--enable-gpl \
\
--disable-static \
--enable-shared \
--disable-runtime-cpudetect \
\
--disable-programs \
\
--disable-doc \
\
--disable-avdevice \
--disable-postproc \
--disable-w32threads \
--disable-os2threads \
\
--enable-decoder=h264_mediacodec \
\
--enable-jni \
--enable-mediacodec \
\
--arch=$ARCH \
--cpu=$CPU  \
--cross-prefix=$CROSS_PREFIX \
--enable-cross-compile \
--sysroot=$SYSROOT \
--target-os=android \
--cc=$CC \
--cxx=$CXX \
--extra-cflags="$EXTRA_CFLAGS" \
--enable-pic \
\
--disable-asm \
\
--enable-neon \

make clean
make -j8
make install

echo "The Compilation of FFmpeg for $CPU is completed."
}

#x86
CPU=i686
ARCH=i686
LOGFILE=`pwd`/Android/log_x86.txt
PREFIX=`pwd`/Android/x86
CC=$TOOLCHAIN/bin/i686-linux-android$API-clang
CXX=$TOOLCHAIN/bin/i686-linux-android$API-clang++
EXTRA_CFLAGS="-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -mstackrealign -D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security   -O2 -DNDEBUG  -fPIC --target=i686-none-linux-android21 --gcc-toolchain=$TOOLCHAIN"
CROSS_PREFIX=$TOOLCHAIN/bin/i686-linux-android-
my_build
```

### **添加 Android 项目需要的 CFLAGS 的技巧**
当 configure 不支持某个参数时，需要主动在 CFLAGS 中添加。

1、新建 Android 的 C++ 项目

2、build release 版本，生成 build.ninja、rules.ninja
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/31.png)

![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/32.png)

3、使用 build.ninja 中的 FLAGS
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/33.png)

```
-g -DANDROID -fdata-sections -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -D_FORTIFY_SOURCE=2 -march=armv7-a -mthumb -Wformat -Werror=format-security   -Oz -DNDEBUG  -fPIC
```

4、使用 rules.ninja 中的参数
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/34.png)

```shell
--target=armv7-none-linux-androideabi21 --gcc-toolchain=/home/weichao/android-sdk/ndk/21.3.6528147/toolchains/llvm/prebuilt/linux-x86_64 --sysroot=/home/weichao/android-sdk/ndk/21.3.6528147/toolchains/llvm/prebuilt/linux-x86_64/sysroot
```

调整路径，修改参数，去掉重复的参数：
```shell
--target=armv7-none-linux-androideabi21 --gcc-toolchain=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64
```

---

# **编译生成 Android 所需的库**

## **给脚本文件设置权限**
```shell
chmod +x build.sh
```

## **执行脚本文件**
```shell
. build.sh
```

## **编译过程**
先打印信息概览：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/35.png)
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/36.png)

编译完成后生成文件：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/37.png)

---

# **编译和使用时可能会遇到的问题**

## **not found**
1、WARNING: /home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/arm-linux-androideabi-pkg-config not found, library detection may fail.
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/38.png)

未发现有什么影响。

2、java.lang.UnsatisfiedLinkError: dlopen failed: library "libclang_rt.ubsan_standalone-aarch64-android.so" not found
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/39.png)

查看 log.txt，有相关信息：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/40.png)

liblog.so 是在：/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/lib/aarch64-linux-android/21/liblog.so

2.1、（失败）按照 clang 的建议，启用 rpath，并设置 ldflags：
```cmake
EXTRA_LDFLAGS="-Wl,-rpath=/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/lib/aarch64-linux-android/21"
```

同时尝试修改 CMakeLists.txt：

* 修改方式 1
```cmkae
add_library(lib_log SHARED IMPORTED)
set_target_properties(lib_log PROPERTIES IMPORTED_LOCATION /home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/lib/aarch64-linux-android/21/liblog.so)
```

* 修改方式 2
```cmkae
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/lib/aarch64-linux-android/21")
```

这样是在 log.txt 中没有报错了，但是在 Android 中依然报错，原因可能是在 CMakeLists.txt 中的修改方式不对，没有链接上。

2.2、找出是哪个配置导致的这个问题：

* --toolchain=clang-usan
    直接去掉这个配置就可以了。

## **unable to create an executable file**
1、/home/weichao/android-ndk-r21d/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang is unable to create an executable file.
C compiler test failed.
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/41.png)

没有信息概览，很明显编译失败了。

查看 log.txt，是 CPU 的值设置错误：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/42.png)

arm64-v8a 的 CPU 的值应该为 armv8-a，修改后：
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/43.png)

## **dlopen failed**
1、java.lang.UnsatisfiedLinkError: dlopen failed: cannot locate symbol "ff_parse_a53_cc" referenced by "/data/app/io.weichao.bwk-D1NGyDX98F4V4LoBM5Qlvg==/lib/arm64/libavcodec.so"...
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/44.png)

* --disable-everything
    很明显是裁减导致的库没有加入 libavcodec.so，关闭 disable everything，验证可以解决这个问题。

应该是播放 MP4 文件时缺少解封装、解码器等，由于功能还在增加中，暂时先关闭 disable everything，以解决这个问题。

## **requires more registers than available**
1、In file included from libswscale/x86/rgb2rgb.c:102:
libswscale/x86/rgb2rgb_template.c:1666:13: error: inline assembly requires more registers than available
            "mov                        %4, %%"FF_REG_a"\n\t"
            ^
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/45.png)

参考[解决Android NDK编译FFmpeg 4.2.2的x86 cpu版时的问题](https://blog.k-res.net/archives/2521.html "https://blog.k-res.net/archives/2521.html")

将 `libavutil/x86/asm.h:75` 中的
```C
#define HAVE_7REGS (ARCH_X86_64 || (HAVE_EBX_AVAILABLE &amp;&amp; HAVE_EBP_AVAILABLE))
```
修改为
```C
#define HAVE_7REGS (ARCH_X86_64)
```

## **has text relocations**
1、E/linker: "/data/app/io.weichao.module_player-Eq7EWqLlZw91dj1uZMG3YA==/lib/x86/libavcodec.so" has text relocations
![](https://weichao-io-1257283924.cos.ap-beijing.myqcloud.com/qldownload/NDK%20r21d%20%2B%20FFmpeg%204.2%20%E7%BC%96%E5%86%99%E8%84%9A%E6%9C%AC%E6%96%87%E4%BB%B6%20%26%20%E7%BC%96%E8%AF%91%E7%94%9F%E6%88%90%20Android%20%E6%89%80%E9%9C%80%E7%9A%84%E5%BA%93/46.png)

参考[ffmpeg安卓x86平台编译错误](https://blog.csdn.net/marco_0631/article/details/73292199?locationNum=5&fps=1 "https://blog.csdn.net/marco_0631/article/details/73292199?locationNum=5&fps=1")

设置：

* --disable-asm

---