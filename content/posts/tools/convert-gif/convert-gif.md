---
title: "在macOS上录制并制作GIF动图"
tags: ["tools"]
date: 2022-09-29T23:26:35+08:00
lastmod: 2022-10-01T12:56:35+08:00
draft: false
---

这是在写blog诞生的副产物：如何录制感兴趣的部分屏幕，再便捷地把它转换成GIF？

**注意！最好是短小的小于30秒录制，不然制作出来的GIF体积太大，不适合网页阅读或者分享到社交网络（一般都有文件最大尺寸要求）**

## 依赖库

使用[Homebrew](https://brew.sh)（没有用过的话请先安装，点开网页有复制一行命令黏贴到终端运行）来安装两个依赖库：*ffmpeg, imagemagick*

```bash
brew install ffmpeg
brew install gifsicle
```

在终端中逐行执行以上命令安装，这可能会花费一些时间。

## 录制屏幕

macOS中拥有一个自带的截图/录制工具，你可以在这里（[在 Mac 上截屏或录制屏幕](https://support.apple.com/zh-cn/guide/mac-help/mh26782/mac)）找到官方的使用说明。

试着玩一下这个内置的工具吧：截图并找到储存的路径、录制屏幕并停止、使用Preview/QuickTime打开录制的视频并裁剪保存。

## 转换成GIF

你可以在桌面上找到刚才录制的视频，它默认的名字会是类似于这样：*Screen Recording 2022-10-01 at 10.12.12 AM.mov*。在终端中操作我们不喜欢带空格的文件名（并且它很长），所以我们将它重命名来方便后面的操作（当然你熟练了可以跳过这一步）。这里我把它重命名成了：*input.mov*

首先我们需要把经过裁剪的录制转换成GIF（但可能文件尺寸较大）

![ffmpeg](../ffmpeg.gif)

```bash
ffmpeg -i $HOME/Desktop/input.mov -r 10 -vf "scale=800:-1, setpts=0.25*PTS" output.gif
```

* -r 代表帧率，对于GIF来说10是一个比较合理的取值，当然具体取多少要看你的录制是比较静态的还是包含大量的运动镜头。

* -vf 是 *video filter* 这里限制了它横向的尺寸为800个像素，竖向自动计算保持横纵比，同时设置setpts加速到原来的4倍（不要的话可以删去，或者设置为1）以减小生成的GIF尺寸。

* -pix_fmt rgb24 这是一个可选参数（但它在我的mac上不生效，不知道其他系统会不会是好的），代表色深，默认是rgb8

## 尺寸优化

```bash
gifsicle -i output.gif -O3 -o optimized.gif
```

* -03 代表调用文件尺寸最优化算法

* --colors 256 这是一个可选参数，也是用来限制色深的，这里我没有加这个参数（因为之前ffmpeg用的是默认的rgb8，colors只有70）

经过我测试最后生成的GIF图是388K，而原本录制的MOV格式则高达11M，体积缩小到了原本的1/25。

## 参考资料

- [OS X Screencast to animated GIF](https://gist.github.com/dergachev/4627207)

- [How can I get ffmpeg to convert a .mov to a .gif?](https://superuser.com/questions/436056/how-can-i-get-ffmpeg-to-convert-a-mov-to-a-gif)

- [Optimize animated GIF size in command-line](https://superuser.com/questions/1107200/optimize-animated-gif-size-in-command-line)


