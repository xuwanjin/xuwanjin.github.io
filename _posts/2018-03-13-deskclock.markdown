---
layout:     post
title:      "DeskClock分析"
subtitle:   "Android DeskClock分析"
date:       2018-03-18 00:10:40
author:     "Mathew"
catalog: true
header-img: "img/post-bg-2017.jpg"
tags:
    - Android
    - DeskClock
---
#DeskClock分析
@[IPO(快速开机), 关机闹钟, 闹钟流程, 闹钟数据库]

[TOC]

### 文件结构

首先认识一下DeskClock这个模块的文件. 
DeskClock进入之后会看到四个页面, AlarmClockFragment(闹钟), ClockFragment(时间), StopwatchFragment(t停表),  TimerFragment(计时器). 他们都继承与DeskClockFragment.


### 闹钟的设置



### 闹钟小部件




### 闹钟播放铃声

```java
Events.java
public static void sendEvent(@StringRes int category, @StringRes int action,
            @StringRes int label) 
```


```java
public void sendEvent(@StringRes int category, @StringRes int action, @StringRes int label)
```

```java
public void sendEvent(String category, String action, String label) 
```

```java
public static void setHighNotificationState(Context context, AlarmInstance instance) 
```

```java
public static void showHighPriorityNotification(Context context, AlarmInstance instance)
```


```java
public void scheduleInstanceStateChange(Context context, Calendar time,
                AlarmInstance instance, int newState)
```


```java
public static Intent createStateChangeIntent(Context context, String tag,
            AlarmInstance instance, Integer state) 
```


```java
public int onStartCommand(Intent intent, int flags, int startId)
```

```java
private void startAlarm(AlarmInstance instance)
```
申请WakeLock， 可以让进程在手机进入睡眠模式也可以持续执行
```
AlarmAlertWakeLock.acquireCpuWakeLock(this);
```


以弹出框的形式弹出通知
```
AlarmNotifications.showAlarmNotification(this, mCurrentAlarm)
```
对电话的监听
```
mInitialCallState = mTelephonyManager.getCallState();
mTelephonyManager.listen(mPhoneStateListener, PhoneStateListener.LISTEN_CALL_STATE);
```


下面的步骤是进行播放通知铃声


```java
AlarmKlaxon
public static void start(Context context, AlarmInstance instance)
```


```
AsyncRingtonePlayer
public void play(Uri ringtoneUri)
```

```java
private void postMessage(int messageCode, Uri ringtoneUri, long delayMillis)
```



```java
AsyncRingtonePlayer
 private Handler getNewHandler()
 getPlaybackDelegate().play(mContext, ringtoneUri)
```


```java
AsyncRingtonePlayer
public boolean play(final Context context, Uri ringtoneUri)
```
```java
mRingtone.play()
```


下面的步骤进行震动


### 闹钟通知




## 关机闹钟

### 关机闹钟的原理



### 关机闹钟的流程
如何设置的, 怎么响应的




### 问题
 每一个闹钟事件是如何存储的
涉及到哪一个类


### 快速开关机(IPO)
快速开关机的宏是
```makefile
MTK_IPOH_SUPPORT = yes
MTK_IPO_SUPPORT = yes
```
设置里面的快速开关机的开关是
在./packages/SettingsProvider/res/values/mtk_defaults.xml 这个路径下面, 默认是打开的
```xml
<bool name="def_ipo_setting">true</bool>
```

在build.prop文件里的ro值是
该值是1的时候, 是打开的状态
```
ro.mtk_ipo_support=1
```
打开之后会在设置里的辅助功能


打开快速开机的宏, 并且让mediaprovider直接启动.+就可以正常使用关机闹钟了


### 相关问题
```
1. 闹钟是如何响应的, 铃声是如何播放的, 没有铃声是如何播放的
2. 闹钟是如何设置, 写到数据库的
3. 

```

##小记
```java
RingtoneManager.getDefaultUri(RingtoneManager.TYPE_ALARM) //content://settings/system/alarm_alert
RingtoneManager.getActualDefaultRingtoneUri(getContext(), RingtoneManager.TYPE_ALARM)//content://media/internal/audio/media/15, 这个是获取真正的URI的
```
 /data/system/users/0/


### 分钟和冒号变小,居上的需求
#### 客户的需求
客户需要这样的效果

![deskclock](/img/apps/deskclock/deskclock_app_widget.png)


#### 实现过程

第一次是想通过布局文件实现, 想到的原因: 第一眼直觉
类似这样的
![deskclock](/img/apps/deskclock/deskclock_app_widget_analysis.png)
就是写一个线性的布局文件然后在里面放两三个TextView, 然后获得时间, 是一个字符串, 然后拆解字符串, 分别显示字符串
不能实现的原因: 等到布局写好了, 发现了这个控件是TextClock, 正是一种特殊的控件, 继承与TextView, 在TextView的方法里就有一个获取时间的, 然后显示时间的, 因此他是一个专门用来显示的时间的控件, 因此这种方法是不可行的

 第二次: 想到上次修改bug,需要修改上午(am), 下午(pm)字体的的大小, 因此一路找到源码. 
 发现最终都是调用了Utils类里面的两个方法get24ModeFormat 和get12ModeFormat 方法. 
 发现这两个的方法返回的都是CharSequence, 一个返回的是"h:mm a"(中间有一个空格), 另一个返回的是"HH:mm". 因此看起来是不大可行的

第三次: 想到的是既然是这个TextClock控件, 那么我们就写两个类去继承这个TextClock类, 然后,改写掉获取时间的方法, 让他们分别去获取时间的分钟和冒号以及时钟. 这样我们就可以显示. 
然而这样有一个问题, 我们去重写的设置TextClock的时间, 发现该方法不能被重写, 准确的说, 该方法被隐藏了, 即onTimeChanged()方法是private类型的. 
这时候我想到了, 干脆把整个TextClock类拿出来, 改写这个控件onTimeChanged()方法. 这样似乎工作量就上来了

于是回到第二个方法, 先看看这个方法

```
/**
 * @param context - context used to get time format string resource
 * @param showAmPm - include the am/pm string if true
 * @return format string for 12 hours mode time
 */
public static CharSequence get12ModeFormat(Context context, boolean showAmPm) {
 // 获取时间格式, 获取的是结果是h:mm a
    String pattern = DateFormat.getBestDateTimePattern(Locale.getDefault(), "hma");
   
   // 是否显示am/pm, 如果不显示, 去掉字符a, 然后去掉空格
    if (!showAmPm) {
        pattern = pattern.replaceAll("a", "").trim();
    }

    // Replace spaces with "Hair Space"
    pattern = pattern.replaceAll(" ", "\u200A");
    // Build a spannable so that the am/pm will be formatted
    // 查找a这个字符在字符串中的位置
    int amPmPos = pattern.indexOf('a');
    if (amPmPos == -1) {
        return pattern;
    }
   // 下面是重要的+
    final Resources resources = context.getResources();
    final float amPmProportion = resources.getFraction(R.fraction.ampm_font_size_scale, 1, 1);
    // 首先是需要一个Spannable, 通俗来说, 这是一个带样式的String.
    final Spannable sp = new SpannableString(pattern);
    sp.setSpan(new RelativeSizeSpan(amPmProportion), amPmPos, amPmPos + 1,
            Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
            // 显示相对大小,从哪到哪, 开闭区间
    sp.setSpan(new StyleSpan(Typeface.NORMAL), amPmPos, amPmPos + 1,
            Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
    sp.setSpan(new TypefaceSpan("sans-serif"), amPmPos, amPmPos + 1,
            Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);

    // Make the font smaller for locales with long am/pm strings.
    // 是否am/pm粗体显示
    if (Utils.isAmPmStringLong()) {
        final float proportion = resources.getFraction(
                R.fraction.reduced_clock_font_size_scale, 1, 1);
        sp.setSpan(new RelativeSizeSpan(proportion), 0, pattern.length(),
                Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
    }
    return sp;
}

```

于是我就仿照写了一个

```
// 我们查找冒号的位置
int colonPos = pattern.indexOf(':');
if (colonPos == -1) {
    return pattern;
}
final float colonProportion = resources.getFraction(R.fraction.ampm_font_size_scale, 1, 1);
// 设置上脚标, 并设置哪些字符设置为脚标
sp.setSpan(new SuperscriptSpan(), colonPos, colonPos + 3,
        Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
sp.setSpan(new StyleSpan(Typeface.NORMAL), amPmPos,  colonPos + 3,
        Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
sp.setSpan(new RelativeSizeSpan(colonProportion), colonPos, colonPos + 3,
        Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);

```

这是最后成定论的的代码. 实际上一开始犯了很多错误.
1. 第一个错误: 一开就把冒号和分钟分开来弄了, 搞得代码很多. 而且还互相冲突. 导致一直不是想要的结果
2. 第二个错误: setSpan的顺序不对, 导致效果总是不对. 应该先设置上脚标, 再设置字体大小的样式
3. 第三个错误: 应该先弄懂这些参数的意思, 再修改

这次将近花了一天半的时间, 感觉时间有点长了. 应该一开始就应该搞懂setspan的参数的含义



#### AppWidget 自定义字体

1. 第一次想法就是xml布局的fontFamily来做, 并没有效果, 百度之后, FontFamily字体只支持monspace 等几种字体
2. 自定义View, 写一个类继承与TextClock. 后来发现. RemoteView只支持几种View, 自定义的并不支持
3. 系统的某些字体是可以使用的, Monospace, sans-serif, courier 是可以使用的, 发现共同的特点是, 他们在fonts.xml都有名字属性, 因此上名字的属性, 就好了.
4. 使用RemoteView的Bitmap方法, 发现更麻烦

### DeskClock 铃声删除, 播放, 没有声音
#### 具体现象
x5921的手机安装了google的DeskClock后发现, 修改一个闹钟的铃声, 选择外部的铃声, 然后删除这个铃声.
然后再增加一个闹钟, 闹钟响应的时候, 发现没有声音. 而同一平台的手机X6061是有声音的. 
1. 刚开始没有觉得这个是正常现象, 毕竟没有音频文件了, 但是对比了一下x6061的现象, 发现两部手机表现不一致
2. 然后觉得x6061的现象是有问题, 还有觉得这是google的app无法修改.


因为是弹出了一个通知, 因此我们看看这个通知的log, 因此抓取出一分比较全面的log文件. 这是一份x5921的log
```
01-02 09:30:20.057   971  1225 V NotificationService: enqueueNotificationInternal: pkg=com.google.android.deskclock id=2147483646 notification=Notification(pri=1 contentView=null vibrate=null sound=null defaults=0x0 flags=0x300 color=0xff5e97f6 category=alarm groupKey=1 vis=PUBLIC)
01-02 09:30:20.062   971   971 D NotificationService: EnqueueNotificationRunnable.run for: 0|com.google.android.deskclock|2147483646|null|10045
01-02 09:30:20.065   971   971 V NotificationService: pkg=com.google.android.deskclock canInterrupt=false intercept=false

```
在这里可以看出来, 971号进程是systemserver进程, 由通知的sound参数可知, 通知并没有传递音频文件的URI参数, 这也是比较符合预期的, 闹钟通知没有声音. 
我们再看看mediaserver进程的log, 几乎所有的音频文件的播放都需要mediaplayer播放(在这里我选择的是一份MP3文件, 音频也比较大, 因此应该是mediaplayer播放). 在这里我们看看该进程的log
```
01-02 09:31:00.776   271  5387 I MediaPlayerService: [create]line:623 Create new client(63) from pid 5080, uid 10045, 
01-02 09:31:00.777   271   551 I MediaPlayerService: [setDataSource]line:1069 [63] setDataSource(content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3)
01-02 09:31:00.784   271   551 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
01-02 09:31:00.786   271  3235 I MediaPlayerService: [disconnect]line:919 disconnect(63) from pid 5080
01-02 09:31:00.786   271  3235 I MediaPlayerService: [~Client]line:908 [63]~Client
01-02 09:31:00.801   271   423 I MediaPlayerService: [create]line:623 Create new client(64) from pid 5080, uid 10045, 
01-02 09:31:00.802   271   271 I MediaPlayerService: [setDataSource]line:1069 [64] setDataSource(content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3)
01-02 09:31:00.806   271   271 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
01-02 09:31:00.808   271  5387 I MediaPlayerService: [disconnect]line:919 disconnect(64) from pid 5080
01-02 09:31:00.809   271  5387 I MediaPlayerService: [~Client]line:908 [64]~Client
01-02 09:31:00.829   271   551 I MediaPlayerService: [create]line:623 Create new client(65) from pid 1107, uid 10030, 
01-02 09:31:00.830   271  3235 I MediaPlayerService: [setDataSource]line:1069 [65] setDataSource(content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3)
01-02 09:31:00.834   271  3235 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
01-02 09:31:00.837   271   423 I MediaPlayerService: [disconnect]line:919 disconnect(65) from pid 1107
01-02 09:31:00.837   271   423 I MediaPlayerService: [~Client]line:908 [65]~Client

```
从log当中出现的Error发现Couldn't open fd for content, 这也是比较符合预期的, 文件的位置已经移动了. 
并且从log的Create new client 字符可以知道曾经三次尝试播放文件MP3文件(也可以从三次的Error看出), 但是三次都未成功, 其中两次是5080号进程, 一次是1107号进程, 上下文的log可以看出, 5080号进程就是该google的DeskClock进程, 而1107进程就是SystemUI的进程. 那么很奇怪的是为什么会出现SystemUI的进程播放文件呢. 

我们在看看正常的情况下, 这个份log
```
12-31 19:02:33.495   292   504 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
12-31 19:03:04.489   292   503 I MediaPlayerService: [create]line:623 Create new client(9) from pid 2270, uid 10045, 
12-31 19:03:04.491   292   504 I MediaPlayerService: [setDataSource]line:1069 [9] setDataSource(content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3)
12-31 19:03:04.500   292   504 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
12-31 19:03:04.506   292   292 I MediaPlayerService: [disconnect]line:919 disconnect(9) from pid 2270
12-31 19:03:04.507   292   292 I MediaPlayerService: [~Client]line:908 [9]~Client
12-31 19:04:01.296   292   503 I MediaPlayerService: [create]line:623 Create new client(10) from pid 2270, uid 10045, 
12-31 19:04:01.298   292   504 I MediaPlayerService: [setDataSource]line:1069 [10] setDataSource(content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3)
12-31 19:04:01.370   292   504 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
12-31 19:04:01.387   292   292 I MediaPlayerService: [disconnect]line:919 disconnect(10) from pid 2270
12-31 19:04:01.388   292   292 I MediaPlayerService: [~Client]line:908 [10]~Client
12-31 19:04:01.442   292   421 I MediaPlayerService: [create]line:623 Create new client(11) from pid 2270, uid 10045, 
12-31 19:04:01.444   292   502 I MediaPlayerService: [setDataSource]line:1069 [11] setDataSource(content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3)
12-31 19:04:01.462   292   502 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
12-31 19:04:01.465   292   503 I MediaPlayerService: [disconnect]line:919 disconnect(11) from pid 2270
12-31 19:04:01.465   292   503 I MediaPlayerService: [~Client]line:908 [11]~Client
12-31 19:04:01.645   292   504 I MediaPlayerService: [create]line:623 Create new client(12) from pid 1089, uid 10030, 
12-31 19:04:01.646   292   292 I MediaPlayerService: [setDataSource]line:1069 [12] setDataSource(content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3)
12-31 19:04:01.667   292   292 E MediaPlayerService: Couldn't open fd for content://com.android.externalstorage.documents/document/primary%3Axuwanjinxuwanjin.MP3
12-31 19:04:01.670   292   421 I MediaPlayerService: [disconnect]line:919 disconnect(12) from pid 1089
12-31 19:04:01.671   292   421 I MediaPlayerService: [~Client]line:908 [12]~Client
12-31 19:04:01.707   292   502 I MediaPlayerService: [create]line:623 Create new client(13) from pid 1089, uid 10030, 
12-31 19:04:01.726   292   503 I MediaPlayerService: [setDataSource]line:1131 [13] setDataSource fd=10 (/system/framework/framework-res.apk), offset=14390324, length=14611
12-31 19:04:01.728   292   503 I MediaPlayerService: [setDataSource_drm_preCheck]line:432 fd=10,path=/system/framework/framework-res.apk
12-31 19:04:01.729   292   503 W MediaPlayerService: setDataSource_drm_preCheck: isFDDrm 1 fd 10 url 0x0
12-31 19:04:01.748   292   503 I MediaPlayerService: [setDataSource]line:1167 player type = 4
12-31 19:04:01.781   292   503 I MediaPlayerService: [setDataSource]line:1175 setDataSource fd=10, offset=14390324, length=14611 done
12-31 19:04:01.798   292   502 I MediaPlayerService: [setVideoSurfaceTexture]line:1230 [13] 
12-31 19:04:01.807   292   292 I MediaPlayerService: [prepareAsync]line:1354 [13] prepareAsync
12-31 19:04:02.054   292  3448 I MediaPlayerService: [notify]line:1720 [13] notify (0xa3c03300, 1, 0, 0, 0x0)
12-31 19:04:02.099   292   421 I MediaPlayerService: [start]line:1372 [13] start
12-31 19:04:02.195   292  3454 I MediaPlayerService: MediaPlayerService::getOMX
12-31 19:04:02.195   292  3454 I MediaPlayerService: MediaPlayerService::getOMX
12-31 19:04:02.682   292  3448 I MediaPlayerService: [notify]line:1720 [13] notify (0xa3c03300, 6, 0, 0, 0x0)
12-31 19:04:02.699   292   502 I MediaPlayerService: [13] getCurrentPosition = 0
12-31 19:04:04.587   292  3448 I MediaPlayerService: [notify]line:1720 [13] notify (0xa3c03300, 6, 0, 0, 0x0)
12-31 19:04:04.595   292   502 I MediaPlayerService: [13] getCurrentPosition = 16
12-31 19:04:06.381   292  3448 I MediaPlayerService: [notify]line:1720 [13] notify (0xa3c03300, 6, 0, 0, 0x0)
12-31 19:04:06.383   292   292 I MediaPlayerService: [13] getCurrentPosition = 3
12-31 19:04:08.192   292  3448 I MediaPlayerService: [notify]line:1720 [13] notify (0xa3c03300, 6, 0, 0, 0x0)
12-31 19:04:08.197   292   292 I MediaPlayerService: [13] getCurrentPosition = 6
12-31 19:04:09.983   292  3448 I MediaPlayerService: [notify]line:1720 [13] notify (0xa3c03300, 6, 0, 0, 0x0)
12-31 19:04:09.987   292   503 I MediaPlayerService: [13] getCurrentPosition = 3
12-31 19:04:11.810   292  3448 I MediaPlayerService: [notify]line:1720 [13] notify (0xa3c03300, 6, 0, 0, 0x0)
12-31 19:04:11.822   292   504 I MediaPlayerService: [13] getCurrentPosition = 22

```

有后面的getCurrentPosition 等字样可以知道, 音频文件正在播放了, 我们看看这个播放的音频文件是什么文件. 从最后一次的new client可以知道, 这个音频文件是 
```
12-31 19:04:01.726   292   503 I MediaPlayerService: [setDataSource]line:1131 [13] setDataSource fd=10 (/system/framework/framework-res.apk), offset=14390324, length=14611

```
这个音频文件竟然是一个app, 这点让我感到很奇怪, 为什么是一个App.  因为这个音频文件就在在这个app里面
再看看是SystemUI要求播放这个文件的, 进程号是1089号. 看这个之前的那一段systemui的进程的log
```
12-31 19:04:01.679  1089  1104 W Ringtone: Remote playback not allowed: java.io.IOException: setDataSource failed.: status=0x80000000
12-31 19:04:01.679  1089  1104 D Ringtone: Problem opening; delegating to remote player
12-31 19:04:01.679  1089  1104 W Ringtone: Neither local nor remote player available when applying playback properties
12-31 19:04:01.680  1089  1104 W Ringtone: Neither local nor remote player available when applying playback properties
```
Ringtone的开始设置资源的时候出现了IO异常, 这点完全符合预期的, 因为没有资源文件了.

期间比对了一下两个不同的分支的的RingtoneManager的代码在这两个函数有很大的不同.
将有没有的分支的代码合入到有声音的那个RingtoneManager. 重新刷机器之后, 表现一致了. 因此两个函数是有问题的. 但具体的原因还是不知道为什么. 继续寻找原因.
```
private static String getSettingForType(int type)
private static String getSettingForType(int type, long simId)
```


因为是Ringtone播放的时候出现了问题， 所以我们再看看Ringtone的play()函数， 也就是看看铃声播放的函数， 这个一个公有的API接口, 因此修改之后需要, 全编译, 这比较麻烦。
#### 说一下最终结果
Ringtone在执行play的操作的时候,如果资源文件不存在了. 
会直接进入到
```java
   public void play() {
   //正常的流程是, 两次调用了play方法, 第二次出现了false为空和mRemotePlayer为空. 第一次则相反
       if (mLocalPlayer != null) {
           // do not play ringtones if stream volume is 0
           // (typically because ringer mode is silent).
           if (mAudioManager.getStreamVolume(
                   AudioAttributes.toLegacyStreamType(mAudioAttributes)) != 0) {
               startLocalPlayer();
           }
       } else if (mAllowRemote && (mRemotePlayer != null)) {
           /// M: Avoid NullPointerException cause by mUri is null.
           final Uri canonicalUri = (mUri == null ? null : mUri.getCanonicalUri());
           final boolean looping;
           final float volume;
           synchronized (mPlaybackSettingsLock) {
               looping = mIsLooping;
               volume = mVolume;
           }
           try {
           //第一次进入了这里, 这里立刻调用了SystemUI的RingtonePlayer去播放了.
           // mRemotePlayer 第一次是允许远程播放的， 所以mRemotePlayer来自于
           // mAudioManager.getRingtonePlayer()， 这个RingtonePlayer实例是来自于
           // SystemServer进程里的AudioService里的mRingtonePlayer， 
           //而mRingtonePlayer是通过setRingtonePlayer方法来赋值的, 
           //setRingtonePlayer 方法是SystemUI的RingtonePlayer的start()方法
           //所以IRingtonePlayer实际上实现是在SystemUI的RingtonePlayer类里面实现了
               mRemotePlayer.play(mRemoteToken, canonicalUri, mAudioAttributes, volume, looping);
           } catch (RemoteException e) {
               if (!playFallbackRingtone()) {
                   Log.w(TAG, "Problem playing ringtone: " + e);
               }
           }
       } else { //没有资源文件的话会直接进入到这里. 执行playFallbackRingtone()方法
		       // 没有资源文件, systemui调用系统的API, 不再允许远程播放了, 最后只有调用这里的方法
           if (!playFallbackRingtone()) {
               Log.w(TAG, "Neither local nor remote playback available");
           }
       }
   }
```

第一次进入到的是第二个条件, 允许有有远程播放, 远程播放的Player不为空, 后接着进入到SystemUI的RingtonePlayer播放文件了. SystemUI会再一次调用API接口, 第二次就不会有远程播放了
```
public void play(IBinder token, Uri uri, AudioAttributes aa, float volume, boolean looping)
        throws RemoteException {
    Log.d(TAG, "RingtonePlayer: mCallback: play:");
    if (LOGD) {
        Log.d(TAG, "play(token=" + token + ", uri=" + uri + ", uid="
                + Binder.getCallingUid() + ")");
    }
    Client client;
    synchronized (mClients) {
        client = mClients.get(token);
        if (client == null) {
            final UserHandle user = Binder.getCallingUserHandle();
            client = new Client(token, uri, user, aa);
            token.linkToDeath(client, 0);
            mClients.put(token, client);
        }
    }
    client.mRingtone.setLooping(looping);
    client.mRingtone.setVolume(volume);
    client.mRingtone.play();   //在这里又再一次转回到SystemUI调用系统的API播放音频文件了
}
```
两次进入到play方法的log
```
12-31 19:04:01.515  2270  3432 D Ringtone: Ringtone: play: mLocalPlayer = null
12-31 19:04:01.515  2270  3432 D Ringtone: Ringtone: play: mAllowRemote = true
12-31 19:04:01.515  2270  3432 D Ringtone: Ringtone: play: mRemotePlayer = android.media.IRingtonePlayer$Stub$Proxy@24fbf85
12-31 19:04:01.680  1089  1104 D Ringtone: Ringtone: play: mLocalPlayer = null
12-31 19:04:01.680  1089  1104 D Ringtone: Ringtone: play: mAllowRemote = false
12-31 19:04:01.680  1089  1104 D Ringtone: Ringtone: play: mRemotePlayer = null
```


下面我们再看看playFallbackRingtone()方法
当系统播放指定的音频文件, 而该音频文件又不存在的时候, 播放的文件是在这个路径下的
frameworks\base\core\res\res\raw\fallbackring.ogg 
具体的代码是在
\frameworks\base\media\java\android\media\下的Ringtone类的
```java
// Ringtone.java
private boolean playFallbackRingtone() {
//这里获取该音频流的音频的音量大小, 如果不为0, 再执行播放操作. 否则有无音频源, 都不播放
   if (mAudioManager.getStreamVolume(AudioAttributes.toLegacyStreamType(mAudioAttributes))
           != 0) {
           //根据要播放的音频的URI判断一下是哪一个默认的音频流.
           // 但是这里的是自己设置的URI, 因此, 他不是任何一个
           //默认的音频流. 返回的值为-1
       int ringtoneType = RingtoneManager.getDefaultType(mUri); 
       // 在这里Ringtone的Type是-1, 正常逻辑返回一个URI, 不为空, 会进入到这里. 
       //然后新的播放资源将会是fallbackring.ogg文件
       //这个文件在frameworks\base\core\res\res\raw\下面. 
       //所以这就是为什么上面的设置音频源为一个App了, 资源文件在这里
       if (RingtoneManager.getActualDefaultRingtoneUri(mContext, ringtoneType) != null) {
           // Default ringtone, try fallback ringtone.
           try {
               AssetFileDescriptor afd = mContext.getResources().openRawResourceFd(
                       com.android.internal.R.raw.fallbackring); //播放的是此ogg文件.
               if (afd != null) {
               // 这次只有本地播放了
                   mLocalPlayer = new MediaPlayer();
                   if (afd.getDeclaredLength() < 0) {
                       mLocalPlayer.setDataSource(afd.getFileDescriptor());
                   } else {
                       mLocalPlayer.setDataSource(afd.getFileDescriptor(),
                               afd.getStartOffset(),
                               afd.getDeclaredLength());
                   }
                   mLocalPlayer.setAudioAttributes(mAudioAttributes);
                   synchronized (mPlaybackSettingsLock) {
                       applyPlaybackProperties_sync();
                   }
                   mLocalPlayer.prepare();
                   startLocalPlayer();
                   afd.close();
                   return true;
               } else {
                   Log.e(TAG, "Could not load fallback ringtone");
               }
           } catch (IOException ioe) {
               destroyLocalPlayer();
               Log.e(TAG, "Failed to open fallback ringtone");
           } catch (NotFoundException nfe) {
               Log.e(TAG, "Fallback ringtone does not exist");
           }
       } else { 
           Log.w(TAG, "not playing fallback for " + mUri);
       }
   }
   return false;
}

```
RingtoneManager.java 获取实际上的默认铃声URI. 

```java
//RingtoneManager.java
public static Uri getActualDefaultRingtoneUri(Context context, int type) {
	// 当URI的资源不存在是, type是-1, 进入到这里
    String setting = getSettingForType(type);
    //当有问题的代码返回的是setting值是null, 因此这里直接返回的默认的URI是NULL
    //没有问题的代码返回的是content://media/internal/audio/media/108
    if (setting == null) return null;
    final String uriString = Settings.System.getStringForUser(context.getContentResolver(),
            setting, context.getUserId());
    Log.i(TAG, "Get actual default ringtone uri= " + uriString);
    return uriString != null ? Uri.parse(uriString) : null;
}
```


没有问题的那段代码
当Type是-1的时候,这里会进入到第一个判断, 这时候会返回Ringtone
```java
//RingtoneManager
private static String getSettingForType(int type) {
     if ((type & TYPE_RINGTONE) != 0) {
         return Settings.System.RINGTONE;
     } else if ((type & TYPE_NOTIFICATION) != 0) {
         return Settings.System.NOTIFICATION_SOUND;
     } else if ((type & TYPE_ALARM) != 0) {
         return Settings.System.ALARM_ALERT;
     // A: Bug_id: add Dual SIM Card ringtone. chenchunyong 20170520 {	
     } else if ((type & TYPE_MESSAGE) != 0) {
         return Settings.System.MESSAGE_SOUND;
     // A: }
     } else {
         return null;
     }
}
```
有问题的那段代码. 
当Type是-1是的时候, 一个类型都不匹配, 所以返回的是null值. 
```java
//RingtoneManager
private static String getSettingForType(int type) {
    if (type == TYPE_RINGTONE) {
        return Settings.System.RINGTONE;
    } else if (type == TYPE_NOTIFICATION) {
        return Settings.System.NOTIFICATION_SOUND;
    } else if (type == TYPE_ALARM) {
        return Settings.System.ALARM_ALERT;
    // A: Bug_id: add Dual SIM Card ringtone. chenchunyong 20170520 {	
    } else if (type == TYPE_MESSAGE) {
        return Settings.System.MESSAGE_SOUND;
    // A: }
    } else {
        return null;
    }
}

```

#### 问题原因总结
添加了一个类型, TYPE_MESSAGE, 然后将getSettingForType方法里的type类型的判断都改成了 == . 
在铃声被删除的情况下, 获取的到Type是-1类型的, 当采用& 他会和所有的都匹配上, 但是只会进入到Ringtone的这个类型里面, 返回的是Ringtone的类型. 
当采用 == 判断的话进入, 所有的条件都不会满足. 返回的是Null.
主要原因是TYPE_MESSAGE的设计不合理, 不应该是3 ,而是2的整数次幂

#### 建议修改方案
将TYPE_MESSAGE这个值改为32, 然后将错误的方法里的==判断条件改为& 然后在判断是否为非零作为条件.
同时将res/res/values/attri.xml文件的关于音频的修改为32. 也就是下面的xml文件. 
其实
在RingtoneManager类里面已经有了这句话
```
// Make sure these are in sync with attrs.xml:
// <attr name="ringtoneType">
```
修改相应的type的类型的值
```xml
<!-- Base attributes available to RingtonePreference. -->
<declare-styleable name="RingtonePreference">
    <!-- Which ringtone type(s) to show in the picker. -->
    <attr name="ringtoneType">
        <!-- Ringtones. -->
        <flag name="ringtone" value="1" />
        <!-- Notification sounds. -->
        <flag name="notification" value="2" />
        <!-- Alarm sounds. -->
        <flag name="alarm" value="4" />
        <!-- All available ringtone sounds. -->
        <flag name="all" value="7" />
    </attr>
    <!-- Whether to show an item for a default sound. -->
    <attr name="showDefault" format="boolean" />
    <!-- Whether to show an item for 'Silent'. -->
    <attr name="showSilent" format="boolean" />
</declare-styleable>
```

说一下在这里情境下声音播放的顺序. 大致的理一下思路
首先是DeskClock正常的调用系统的API的Ringtone的Play()方法,  在第一次的播放里参数mAllowRemote为true, 允许远程播放, 同时mRemotePlayer不为空, 这时候mRemotePlayer也调用play方法, 这时候会有进程间的调用, 进入到SystemUI进程调用到RingtonePlayer类的mCallback的play方法. mRingtone也是调用系统的API方法, 这个时候allowRemote参数为false. 这次会进入到第三个条件里面执行playFallbackRingtone. 在这里会播放系统的默认声音. 
总的就是DeskClock调用系统API--->远程播放进入到SystemUI--->SystemUI调用系统API
当然最终还是MediaServer播放了该音频



### DeskClock 设置闹钟铃声	停止运行

#### log文件
先给一下log
```
01-01 14:07:41.868407 29225 29225 D Surface : Surface::allocateBuffers(this=0x7943bec000)
01-01 14:07:41.871032 29225 29245 I OpenGLRenderer: Initialized EGL, version 1.4
01-01 14:07:41.871199 29225 29245 D OpenGLRenderer: Swap behavior 2
01-01 14:07:41.873206 29225 29245 D OpenGLRenderer: Created EGL context (0x795243c900)
01-01 14:07:41.874719 29225 29245 D OpenGLRenderer: ProgramCache.init: enable enhancement 1
01-01 14:07:41.874784 29225 29245 I OpenGLRenderer: Already had program atlas...
01-01 14:07:41.874932 29225 29245 D OpenGLRenderer: Initializing program cache from 0x0, size = -1
01-01 14:07:41.875458 29225 29245 D Surface : Surface::connect(this=0x7943bec000,api=1)
01-01 14:07:41.876372   309   434 I BufferQueueProducer: [com.android.deskclock/com.android.deskclock.ringtone.RingtonePickerActivity#0](this:0x79bd7df800,id:1645,api:1,p:29225,c:309) connect(P): api=1 producer=(29225:com.android.deskclock) producerControlledByApp=true
01-01 14:07:41.883029 29225 29245 D mali_winsys: EGLint new_window_surface(egl_winsys_display *, void *, EGLSurface, EGLConfig, egl_winsys_surface **, egl_color_buffer_format *, EGLBoolean) returns 0x3000
01-01 14:07:41.906787   304   346 I vendor.mediatek.hardware.power@1.1-impl: mtkPowerHint hint:12, data:2000
01-01 14:07:41.909282   868 29416 D AES     : onEndOfErrorDumpThread: system_app_crash Process: com.android.deskclock
01-01 14:07:41.909282   868 29416 D AES     : PID: 29225
01-01 14:07:41.909282   868 29416 D AES     : Flags: 0x38c83e45
01-01 14:07:41.909282   868 29416 D AES     : Package: com.android.deskclock v26 (8.0.0)
01-01 14:07:41.909282   868 29416 D AES     : Foreground: Yes
01-01 14:07:41.909282   868 29416 D AES     : Build: alps/x6161_public_64/x6161_public_64:8.0.0/O00623/1511816327:user/release-keys
01-01 14:07:41.909282   868 29416 D AES     : 
01-01 14:07:41.909282   868 29416 D AES     : java.lang.RuntimeException: An error occurred while executing doInBackground()
01-01 14:07:41.909282   868 29416 D AES     : 	at android.os.AsyncTask$3.done(AsyncTask.java:353)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:383)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.concurrent.FutureTask.setException(FutureTask.java:252)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.concurrent.FutureTask.run(FutureTask.java:271)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1162)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:636)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.lang.Thread.run(Thread.java:764)
01-01 14:07:41.909282   868 29416 D AES     : Caused by: java.util.ConcurrentModificationException
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.ArrayList$Itr.next(ArrayList.java:860)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.Collections$UnmodifiableCollection$1.next(Collections.java:1084)
01-01 14:07:41.909282   868 29416 D AES     : 	at com.android.deskclock.ringtone.RingtoneLoader.loadInBackground(RingtoneLoader.java:93)
01-01 14:07:41.909282   868 29416 D AES     : 	at com.android.deskclock.ringtone.RingtoneLoader.loadInBackground(RingtoneLoader.java:61)
01-01 14:07:41.909282   868 29416 D AES     : 	at android.content.AsyncTaskLoader.onLoadInBackground(AsyncTaskLoader.java:315)
01-01 14:07:41.909282   868 29416 D AES     : 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:69)
01-01 14:07:41.909282   868 29416 D AES     : 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:64)
01-01 14:07:41.909282   868 29416 D AES     : 	at android.os.AsyncTask$2.call(AsyncTask.java:333)
01-01 14:07:41.909282   868 29416 D AES     : 	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
01-01 14:07:41.909282   868 29416 D AES     : 	... 3 more
01-01 14:07:41.909282   868 29416 D AES     :  29225
01-01 14:07:41.909451   868 29416 W AES     : Exception Log handling...
01-01 14:07:41.912161 29225 29245 D OpenGLRenderer: CacheTexture 3 upload: x, y, width height = 0, 0, 224, 500
01-01 14:07:41.912309 29225 29245 D OpenGLRenderer: ProgramCache.generateProgram: 0
01-01 14:07:41.916851 29225 29245 D OpenGLRenderer: ProgramCache.generateProgram: 34359738368
01-01 14:07:41.918570 29225 29245 D OpenGLRenderer: ProgramCache.generateProgram: 562984313159683
01-01 14:07:41.920823   868 29416 D AES     : ExceptionLog: notify aed
01-01 14:07:41.920964   868 29416 D AES     :     process : com.android.deskclock
01-01 14:07:41.920989   868 29416 D AES     :      module : com.android.deskclock v26 (8.0.0)
01-01 14:07:41.920989   868 29416 D AES     : 
01-01 14:07:41.921005   868 29416 D AES     :       cause : system_app_crash
01-01 14:07:41.921022   868 29416 D AES     :       pid : 29225
01-01 14:07:41.921219 29225 29245 D OpenGLRenderer: ProgramCache.generateProgram: 562984313159681
01-01 14:07:41.922633   868 29416 D AEE_LIBAEE: shell: raise_exp(4, 29225, -1361051648, com.android.deskclock, 0x0x7935141b40, 0x0x0)
01-01 14:07:41.923795 29225 29245 D OpenGLRenderer: ProgramCache.generateProgram: 562949953421313
01-01 14:07:41.924216   868 29416 D AEE_LIBAEE: com.mtk.aee.aed_64shell: connected with AED OK
01-01 14:07:41.925911 29225 29245 D OpenGLRenderer: ProgramCache.generateProgram: 562949962858561
01-01 14:07:41.928362 29225 29245 D OpenGLRenderer: ProgramCache.generateProgram: 562949958664257
01-01 14:07:41.930695   514   514 D AEE_AED : $===AEE===AEE===AEE===$
01-01 14:07:41.930888   514   514 D AEE_AED : p 1 poll events 1 revents 1
01-01 14:07:41.932016   514   514 D AEE_AED : PPM cpu cores:4, online:4
01-01 14:07:41.933788   514   514 D AEE_AED : aed_main_fork_worker: generator 0x72c0a2fe60, worker 0x7fd541bb58, recv_fd 10
01-01 14:07:41.938722 29418 29418 D AEE_AED : read success, handling msg (Ind, AE_IND_EXP_RAISED)
01-01 14:07:41.939757 29418 29418 I AEE_AED : [preset_info] pid: 29225, tid: -1361051648, name: UNKNOWN  >>> com.android.deskclock <<<
01-01 14:07:41.940316 29418 29418 D AEE_AED : u:r:system_app:s0
01-01 14:07:41.940522 29418 29418 V AEE_AED : dashboard_record_update() : rec->module = com.android.deskclock 
01-01 14:07:41.940561 29418 29418 V AEE_AED : Update record[2] 
01-01 14:07:41.940581 29418 29418 D AEE_AED :  i, Cls,	 count,	 last_time,	 module  
01-01 14:07:41.940598 29418 29418 D AEE_AED : ==================================================================== 
01-01 14:07:41.940613 29418 29418 D AEE_AED :  0, 11,	    1,	 1483228856,	 system_server 
01-01 14:07:41.940630 29418 29418 D AEE_AED :  1, 11,	    8,	 1483274451,	 com.google.android.music:main 
01-01 14:07:41.940645 29418 29418 D AEE_AED :  2,  4,	    1,	 1483276061,	 com.android.deskclock 
01-01 14:07:41.940660 29418 29418 D AEE_AED :  3, -1,	    0,	 0,	  
01-01 14:07:41.940674 29418 29418 D AEE_AED :  4, -1,	    0,	 0,	  
01-01 14:07:41.940688 29418 29418 D AEE_AED :  5, -1,	    0,	 0,	  
01-01 14:07:41.940701 29418 29418 D AEE_AED :  6, -1,	    0,	 0,	  
01-01 14:07:41.940717 29418 29418 D AEE_AED :  7, -1,	    0,	 0,	  
01-01 14:07:41.940732 29418 29418 D AEE_AED : Skip for Exp level'0'
01-01 14:07:41.941153   868 29416 D AEE_LIBAEE: shell: got the request (cmd:Ind,AE_IND_LOG_CLOSE)
01-01 14:07:41.941241   868 29416 D AEE_LIBAEE: shell: Got session close ind from AED
01-01 14:07:41.942467   514   514 D AEE_AED : clear ppm settings
01-01 14:07:41.948770   514   514 D AEE_AED : $===AEE===AEE===AEE===$
01-01 14:07:41.957802   304   346 I vendor.mediatek.hardware.power@1.1-impl: mtkPowerHint hint:12, data:2000
01-01 14:07:41.966591   304   346 I vendor.mediatek.hardware.power@1.1-impl: notifyAppState pack:com.android.deskclock, act:com.android.deskclock.DeskClock, pid:29225, state:1
01-01 14:07:41.967419   304   326 I powerd  : [powerd_req] POWER_MSG_NOTIFY_STATE
01-01 14:07:41.967540   304   326 I libPerfService: perfUserGetCapability - cmd:7, id:0
01-01 14:07:41.967587   304   326 I libPerfService: perfUserGetCapability - value:-1
01-01 14:07:41.986568 29225 29225 I cj      : strFormat == dd/MM/yyyy/ndateFormat == dd/MM/yyyy

```

#### 异常代码段
log显示的代码出现的问题的
```java
final List<ItemAdapter.ItemHolder<Uri>> itemHolders = new ArrayList<>(itemCount);

// Add the item holder for the Music heading.
itemHolders.add(new HeaderHolder(R.string.your_sounds));

// Add an item holder for each custom ringtone and also cache a pretty name.
/// M: [ALPS03552747] Access  mCustomRingtones only if it is not null@{
if (mCustomRingtones != null) {
    for (CustomRingtone ringtone : mCustomRingtones) {
        itemHolders.add(new CustomRingtoneHolder(ringtone));  //这行出现了问题
    }
}
// Add an item holder for the "Add new" music ringtone.
```


#### 解决方案是
更改遍历的方式
```java
if (mCustomRingtones != null) {
    /*  for (CustomRingtone ringtone : mCustomRingtones) {
        itemHolders.add(new CustomRingtoneHolder(ringtone));

    }*/
    for(int x = 0; x < mCustomRingtones.size(); x ++){
        itemHolders.add(new CustomRingtoneHolder(mCustomRingtones.get(x)));
    }
}
```

## 参考文献
1. [Android RTC 自下而上分析](http://www.embedu.org/column/Column468.htm)
2. [ Android RTC 自下往上浅析](http://blog.csdn.net/crycheng/article/details/7802502)
3. [N上关机闹钟铃声问题](https://onlinesso.mediatek.com/Pages/FAQ.aspx?List=SW&FAQID=FAQ19292)
4. [Android源码之DeskClock (一)](http://blog.csdn.net/l2show/article/details/46709863)
5. [Android源码之DeskClock (二)](http://blog.csdn.net/l2show/article/details/46722999)
6. [SQLite 3.7的新特性.db-shm和.db-wal](http://www.jianshu.com/p/9fbbbf32ccde)
7. [Android时钟应用的定时框架分析](http://blog.csdn.net/xu_fu/article/details/39525297)
8. [Android Alarm自上而下 调试浅析](http://www.cnblogs.com/chenlong-50954265/p/5437685.html)
9. [Full-Disk Encryption](https://source.android.com/security/encryption/full-disk.html)
10. [Wakelock解析](http://souly.cn/%E6%8A%80%E6%9C%AF%E5%8D%9A%E6%96%87/2015/11/05/wakelock%E8%A7%A3%E6%9E%90/)
11. [Android 4.2 关于GlowPadView的说明 ](http://blog.csdn.net/yihongyuelan/article/details/14000363)
12. [Android中各种Span的用法 ](http://blog.csdn.net/qq_16430735/article/details/50427978)
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()
[]()