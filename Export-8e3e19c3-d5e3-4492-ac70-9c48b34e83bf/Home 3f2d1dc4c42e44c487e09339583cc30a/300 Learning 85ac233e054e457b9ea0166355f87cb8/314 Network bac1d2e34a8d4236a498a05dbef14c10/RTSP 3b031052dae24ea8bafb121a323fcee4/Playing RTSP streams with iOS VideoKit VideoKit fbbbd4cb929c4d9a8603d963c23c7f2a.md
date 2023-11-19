# Playing RTSP streams with iOS VideoKit | VideoKit

As we all know iOS SDK does not support playing RTSP streams natively. But if you want to play RTSP streams in your applications, you can use iOS VideoKit for doing that. It’s as easy as adding a few lines of code to play RTSP streams using iOS VideoKit. We will come to that in the next paragraphs but before, let’s check out how RTSP works a little bit.

RTSP is used to establish and control media sessions between client and server. The transmission of streaming data is not a task of RTSP protocol. Most RTSP servers use the Real-time Transport Protocol (RTP) in conjunction with Real-time Control Protocol (RTCP) for media stream delivery. RTSP 2.0 was published as RFC 7826 in 2016 by IETF.

While similar in some ways to HTTP, RTSP defines control sequences useful in controlling multimedia playback. While HTTP is stateless, RTSP has state; an identifier is used when needed to track concurrent sessions. Like HTTP, RTSP uses TCP to maintain an end-to-end connection as default.

Here is a sample wireshark capture of an RTSP video streaming sequence, which may give you an idea of what’s going on while establishing an RTSP connection.

[Screen Shot 2017-02-05 at 13.15.27](https://iosvideokit.com/wp-content/uploads/2017/02/Screen-Shot-2017-02-05-at-13.15.27.png)

![Playing%20RTSP%20streams%20with%20iOS%20VideoKit%20VideoKit%20fbbbd4cb929c4d9a8603d963c23c7f2a/Screen-Shot-2017-02-05-at-13.15.27.png](Playing%20RTSP%20streams%20with%20iOS%20VideoKit%20VideoKit%20fbbbd4cb929c4d9a8603d963c23c7f2a/Screen-Shot-2017-02-05-at-13.15.27.png)

First of all it starts with an OPTIONS request which returns the request types the server will accept.

Such as DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE.

Then a DESCRIBE request includes an RTSP URL (rtsp://…), and the type of reply data that can be handled. This reply includes the presentation description, typically in Session Description Protocol (SDP) format. Among other things, the presentation description lists the media streams controlled with the aggregate URL. In the typical case, there is one

media stream each for audio and video.

After 200 OK response for the DESCRIBE request, a SETUP request is sent which specifies how a single media stream must be transported. This must be done before a PLAY request

is sent. The request contains the media stream URL and a transport specifier. This specifier typically includes a local port for receiving RTP data (audio or video), and another for

RTCP data (meta information). The server reply usually confirms the chosen parameters, and fills in the missing parts, such as the server’s chosen ports. Each media stream must

be configured using SETUP before an aggregate play request may be sent.

After 200 OK response for the SETUP request, a PLAY request is sent to play media stream. Play requests can be stacked by sending multiple PLAY requests. The URL may be the aggregate

URL (to play all media streams), or a single media stream URL (to play only that stream). A range can be specified. If no range is specified, the stream is played from the beginning

and plays to the end, or, if the stream is paused, it is resumed at the point it was paused.

Now let’s talk about the transport options that iOS VideoKit supports.

TCP is the default transport layer for RTSP protocol in iOS VideoKit unless any decoder options are given. But if you want to use UDP, UDP Multicast, or HTTP those are also available

as configuration options.

You can set VKDECODER_OPT_KEY_RTSP_TRANSPORT key to following values :

TCP => VKDECODER_OPT_VALUE_RTSP_TRANSPORT_TCP (default)

UDP => VKDECODER_OPT_VALUE_RTSP_TRANSPORT_UDP

UDP Multicast => VKDECODER_OPT_VALUE_RTSP_TRANSPORT_UDP_MULTICAST

HTTP => VKDECODER_OPT_VALUE_RTSP_TRANSPORT_HTTP

Setting the transport option for RTSP streaming is really easy in iOS VideoKit.

Here is an example of creating transport options:

`NSDictionary *options = [NSDictionary dictionaryWithObject:VKDECODER_OPT_VALUE_RTSP_TRANSPORT_UDP forKey:VKDECODER_OPT_KEY_RTSP_TRANSPORT];`

And then you give this options to Channel object :

`Channel *chanBigBuckBunny = [Channel channelWithName:@"My Channel" addr:@"rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov" description:@"rtsp sample stream" localFile:NO options:options];`

That’s it! In iOS VideoKit with just adding 2 lines of code you can easily set options for RTSP streaming and start playing video without thinking about protocol overhead. iOS VideoKit

does the rest for you…

[Tweet](https://twitter.com/intent/tweet?original_referer=https%3A%2F%2Fiosvideokit.com%2F&ref_src=twsrc%5Etfw%7Ctwcamp%5Ebuttonembed%7Ctwterm%5Eshare%7Ctwgr%5E&text=Playing%20RTSP%20streams%20with%20iOS%20VideoKit&url=https%3A%2F%2Fiosvideokit.com%2F2017%2F02%2F05%2Frtsp-ffmpeg-ios-videokit%2F)