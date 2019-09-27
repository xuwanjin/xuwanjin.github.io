---
title: 记一次 Monkey Calendar 解决的过程
date: 2019-09-27 21:49:25
tags:
    - Calendar
    - Monkey
    - Android
---

最近遇到了一个 Monkey 的bug.
<!-- more -->
# 记一次 Monkey Calendar 解决的过程
## 背景
最近遇到了一个 Monkey 的bug.
```
07-12 21:40:06.834 29668 29687 E AndroidRuntime: FATAL EXCEPTION: AsyncTask #1
07-12 21:40:06.834 29668 29687 E AndroidRuntime: Process: com.android.calendar, PID: 29668
07-12 21:40:06.834 29668 29687 E AndroidRuntime: java.lang.RuntimeException: An error occurred while executing doInBackground()
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.AsyncTask$3.done(AsyncTask.java:325)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:354)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.setException(FutureTask.java:223)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:242)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.lang.Thread.run(Thread.java:761)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: Caused by: java.lang.SecurityException: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{4d13031 29668:com.android.calendar/u0a16} (pid=29668, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1684)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1637)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.app.ActivityManagerProxy.getContentProvider(ActivityManagerNative.java:4199)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.app.ActivityThread.acquireProvider(ActivityThread.java:5478)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.app.ContextImpl$ApplicationContentResolver.acquireUnstableProvider(ContextImpl.java:2239)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.ContentResolver.acquireUnstableProvider(ContentResolver.java:1520)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.ContentResolver.query(ContentResolver.java:518)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:64)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:56)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.AsyncTaskLoader.onLoadInBackground(AsyncTaskLoader.java:312)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:69)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:66)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.AsyncTask$2.call(AsyncTask.java:305)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:237)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	... 3 more

```
 显示了 Calendar 出现了异常错误的状况, 但是在异常调用栈里却没有显示任何在 Calendar 代码的地方. 这个刚开始很困惑, 后来还是解决了. 以下是记录了解决的过程.



## 分析过程
### 第一份错误 log
``` 
07-12 21:37:56.706  1095  1194 D SubscriptionController: [getActiveSubInfoList] Sub Controller not ready
07-12 21:38:06.973  1095  1095 D RILJ    : [3803]> RIL_REQUEST_GET_ACTIVITY_INFO [SUB0]
07-12 21:38:06.973  1095  1095 D RilRequest: [3803]< RIL_REQUEST_GET_ACTIVITY_INFO error: com.android.internal.telephony.CommandException: RADIO_NOT_AVAILABLE ret=
07-12 21:38:24.462  1095  1107 D QtiSubscriptionController: getPhoneId, received dummy subId 2147483643
07-12 21:38:34.886  1095  1108 D SubscriptionController: [getActiveSubInfoList] Sub Controller not ready
07-12 21:38:36.991  1095  1095 D RILJ    : [3804]> RIL_REQUEST_GET_ACTIVITY_INFO [SUB0]
07-12 21:38:36.991  1095  1095 D RilRequest: [3804]< RIL_REQUEST_GET_ACTIVITY_INFO error: com.android.internal.telephony.CommandException: RADIO_NOT_AVAILABLE ret=
07-12 21:38:51.317  1095  1108 D QtiSubscriptionController: getPhoneId, received dummy subId 2147483643
07-12 21:39:14.804  1095  1194 D QtiSubscriptionController: [getSlotId]- subId invalid
07-12 21:39:14.805 22214 22214 D SubscriptionManager: [getSubId]- fail
07-12 21:39:24.953  1095  1095 I SST     : Auto time state changed
07-12 21:39:28.146 13864 29559 E AndroidRuntime: FATAL EXCEPTION: AsyncTask #1
07-12 21:39:28.146 13864 29559 E AndroidRuntime: Process: com.android.calendar, PID: 13864
07-12 21:39:28.146 13864 29559 E AndroidRuntime: java.lang.RuntimeException: An error occurred while executing doInBackground()
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.AsyncTask$3.done(AsyncTask.java:325)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:354)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.setException(FutureTask.java:223)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:242)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at java.lang.Thread.run(Thread.java:761)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: Caused by: java.lang.SecurityException: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{3121c69 13864:com.android.calendar/u0a16} (pid=13864, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1684)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1637)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.app.ActivityManagerProxy.getContentProvider(ActivityManagerNative.java:4199)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.app.ActivityThread.acquireProvider(ActivityThread.java:5478)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.app.ContextImpl$ApplicationContentResolver.acquireUnstableProvider(ContextImpl.java:2239)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.ContentResolver.acquireUnstableProvider(ContentResolver.java:1520)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.ContentResolver.query(ContentResolver.java:518)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:64)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:56)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.AsyncTaskLoader.onLoadInBackground(AsyncTaskLoader.java:312)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:69)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:66)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.AsyncTask$2.call(AsyncTask.java:305)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:237)
07-12 21:39:28.146 13864 29559 E AndroidRuntime: 	... 3 more
07-12 21:39:51.844  1095  1095 I SST     : Auto time state changed
07-12 21:39:51.852  1095  1095 D SST     : Reverting to NITZ Time: mSavedTime=0 mSavedAtTime=0
07-12 21:39:57.991  1095  1095 I SST     : Auto time state changed
07-12 21:40:00.072  1095  1095 I SST     : Auto time state changed
07-12 21:40:00.073  1095  1095 D SST     : Reverting to NITZ Time: mSavedTime=0 mSavedAtTime=0
07-12 21:40:06.834 29668 29687 E AndroidRuntime: FATAL EXCEPTION: AsyncTask #1
07-12 21:40:06.834 29668 29687 E AndroidRuntime: Process: com.android.calendar, PID: 29668
07-12 21:40:06.834 29668 29687 E AndroidRuntime: java.lang.RuntimeException: An error occurred while executing doInBackground()
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.AsyncTask$3.done(AsyncTask.java:325)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:354)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.setException(FutureTask.java:223)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:242)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.lang.Thread.run(Thread.java:761)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: Caused by: java.lang.SecurityException: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{4d13031 29668:com.android.calendar/u0a16} (pid=29668, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1684)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1637)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.app.ActivityManagerProxy.getContentProvider(ActivityManagerNative.java:4199)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.app.ActivityThread.acquireProvider(ActivityThread.java:5478)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.app.ContextImpl$ApplicationContentResolver.acquireUnstableProvider(ContextImpl.java:2239)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.ContentResolver.acquireUnstableProvider(ContentResolver.java:1520)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.ContentResolver.query(ContentResolver.java:518)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:64)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:56)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.AsyncTaskLoader.onLoadInBackground(AsyncTaskLoader.java:312)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:69)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:66)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at android.os.AsyncTask$2.call(AsyncTask.java:305)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:237)
07-12 21:40:06.834 29668 29687 E AndroidRuntime: 	... 3 more
07-12 21:40:17.006  1095  1095 I SST     : Auto time state changed
07-12 21:41:59.926  1095  1095 D RILJ    : [3805]> RIL_REQUEST_GET_ACTIVITY_INFO [SUB0]
07-12 21:41:59.926  1095  1095 D RilRequest: [3805]< RIL_REQUEST_GET_ACTIVITY_INFO error: com.android.internal.telephony.CommandException: RADIO_NOT_AVAILABLE ret=
07-12 21:42:00.247  1095  1095 D RILJ    : [3806]> RIL_REQUEST_GET_ACTIVITY_INFO [SUB0]
07-12 21:42:00.248  1095  1095 D RilRequest: [3806]< RIL_REQUEST_GET_ACTIVITY_INFO error: com.android.internal.telephony.CommandException: RADIO_NOT_AVAILABLE ret=
07-12 21:42:03.983  1095  1195 D QtiSubscriptionController: getPhoneId, received dummy subId 2147483643
07-12 21:42:04.020  1095  1095 D RILJ    : [3807]> RIL_REQUEST_GET_ACTIVITY_INFO [SUB0]
07-12 21:42:04.021  1095  1095 D RilRequest: [3807]< RIL_REQUEST_GET_ACTIVITY_INFO error: com.android.internal.telephony.CommandException: RADIO_NOT_AVAILABLE ret=
07-12 21:42:04.118  1095  1095 D RILJ    : [3808]> RIL_REQUEST_GET_ACTIVITY_INFO [SUB0]
07-12 21:42:04.119  1095  1095 D RilRequest: [3808]< RIL_REQUEST_GET_ACTIVITY_INFO error: com.android.internal.telephony.CommandException: RADIO_NOT_AVAILABLE ret=
```
在这份log当中, Calendar进程出现了连续两次错误. 在进程 13864 挂掉之前, 搜整份log的时候, 关于进程 13864, 并没有其他的收获. 
感觉这个发生的异常只是 logcat 里的缓存. 刚好被 logcat 抓取了.
### 第二份 log  
同时这错误发生的前后, 都有关于时间的log, 似乎, 这个错误和时间有关系. 但是我们还不能确定.
以下的log, 是一份完整的. 在 进程calendar挂掉之前, calendar的进程
``` log
07-13 07:39:24.246 13864 13864 I am_on_paused_called: [0,com.android.calendar.RequestPermissionsActivity,handlePauseActivity]
07-13 07:39:24.263   729 20384 I am_restart_activity: [0,99129693,529,com.android.packageinstaller/.permission.ui.GrantPermissionsActivity]
07-13 07:39:24.240 13884 13884 I auditd  : type=1400 audit(0.0:1775): avc: denied { read } for comm="RenderThread" name="gpuclk" dev="sysfs" ino=10638 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:sysfs:s0 tclass=file permissive=0
07-13 07:39:24.240 13884 13884 W RenderThread: type=1400 audit(0.0:1775): avc: denied { read } for name="gpuclk" dev="sysfs" ino=10638 scontext=u:r:untrusted_app:s0:c512,c768 tcontext=u:object_r:sysfs:s0 tclass=file permissive=0
07-13 07:39:24.272   729   744 W ActivityManager: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{3121c69 13864:com.android.calendar/u0a16} (pid=13864, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
07-13 07:39:24.273   729   756 I sysui_count: [window_time_0,0]
07-13 07:39:24.273   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:24.274   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:24.275 13864 29525 W AsyncQuery: java.lang.SecurityException: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{3121c69 13864:com.android.calendar/u0a16} (pid=13864, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
07-13 07:39:24.360 14419 14419 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@bccc6f7
07-13 07:39:24.360 14419 14419 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@4da2964
07-13 07:39:24.361   729   778 I sysui_action: [322,220]
07-13 07:39:24.405   729 17327 I am_finish_activity: [0,99129693,529,com.android.packageinstaller/.permission.ui.GrantPermissionsActivity,app-request]
07-13 07:39:24.408   729 17327 I wm_task_moved: [529,1,17]
07-13 07:39:24.420   729 17327 I am_focused_activity: [0,com.android.calendar/.RequestPermissionsActivity,finishActivity adjustFocus]
07-13 07:39:24.420   729 17327 D ActivityTrigger: ActivityTrigger activityPauseTrigger 
07-13 07:39:24.421   729 17327 I am_pause_activity: [0,99129693,com.android.packageinstaller/.permission.ui.GrantPermissionsActivity]
07-13 07:39:24.421   729 17327 D PowerManagerService: acquireWakeLockInternal: lock=840646, flags=0x1, tag="*launch*", ws=WorkSource{10007}, uid=1000, pid=729
07-13 07:39:24.421   729 17327 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x1
07-13 07:39:24.422   729 17327 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61612185 (in 10375 ms)
07-13 07:39:24.422   729 17327 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x1, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:24.428   729 30174 D PowerManagerService: updateWakeLockWorkSourceInternal: lock=840646 [*launch*], ws=WorkSource{10016}
07-13 07:39:24.441   729 30174 I am_resume_activity: [0,223991823,529,com.android.calendar/.RequestPermissionsActivity]
07-13 07:39:24.450   729   778 I sysui_action: [320,2]
07-13 07:39:24.451   729   778 I sysui_action: [319,309]
07-13 07:39:24.455   729  1002 I am_finish_activity: [0,223991823,529,com.android.calendar/.RequestPermissionsActivity,app-request]
07-13 07:39:24.463   729  1002 I wm_task_moved: [523,1,17]
07-13 07:39:24.469   729  1002 I am_focused_activity: [0,com.android.settings/.Settings$DateTimeSettingsActivity,finishActivity adjustFocus]
07-13 07:39:24.469   729  1002 D ActivityTrigger: ActivityTrigger activityPauseTrigger 
07-13 07:39:24.470   729  1002 I am_pause_activity: [0,223991823,com.android.calendar/.RequestPermissionsActivity]
07-13 07:39:24.470   729  1002 D PowerManagerService: acquireWakeLockInternal: lock=840646, flags=0x1, tag="*launch*", ws=WorkSource{10016}, uid=1000, pid=729
07-13 07:39:24.470   729  1002 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x1
07-13 07:39:24.471   729  1002 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61612185 (in 10326 ms)
07-13 07:39:24.471   729  1002 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x1, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:24.474   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:24.474   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:24.487   729 17317 D PowerManagerService: updateWakeLockWorkSourceInternal: lock=840646 [*launch*], ws=WorkSource{1000}
07-13 07:39:24.526   729 17321 D WindowManager: relayoutVisibleWindow: Window{edb4209 u0 com.android.settings/com.android.settings.Settings$DateTimeSettingsActivity EXITING} mAnimatingExit=true, mRemoveOnExit=false, mDestroying=false
07-13 07:39:24.544   729 17317 I am_resume_activity: [0,69092980,523,com.android.settings/.Settings$DateTimeSettingsActivity]
07-13 07:39:24.574   729   756 I am_destroy_activity: [0,99129693,529,com.android.packageinstaller/.permission.ui.GrantPermissionsActivity,finish-imm]
07-13 07:39:24.585   729   756 I am_destroy_activity: [0,59729483,529,com.android.calendar/.AllInOneActivity,finish-imm]
07-13 07:39:24.606   729   755 I dvm_lock_sample: [system_server,0,android.bg,32,ActivityManagerService.java,2434,ActivityStackSupervisor.java,3915,6]
07-13 07:39:24.609 13864 13864 I art     : Starting a blocking GC Explicit
07-13 07:39:24.615 22214 22214 I sysui_view_visibility: [38,100]
07-13 07:39:24.622 22214 22214 I am_on_resume_called: [0,com.android.settings.Settings$DateTimeSettingsActivity,RESUME_ACTIVITY]
07-13 07:39:24.634   729 17327 D PowerManagerService: releaseWakeLockInternal: lock=840646 [*launch*], flags=0x0
07-13 07:39:24.634   729 17327 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x0
07-13 07:39:24.634   729 17327 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61612185 (in 10162 ms)
07-13 07:39:24.635   729 17327 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x0, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:24.635   729 17327 D PowerManagerService: Releasing suspend blocker "PowerManagerService.WakeLocks".
07-13 07:39:24.674   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:24.674   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:24.684 13864 13864 I art     : Explicit concurrent mark sweep GC freed 5573(411KB) AllocSpace objects, 0(0B) LOS objects, 63% free, 7MB/19MB, paused 551us total 74.492ms
07-13 07:39:24.689 13864 13864 I art     : Starting a blocking GC Explicit
07-13 07:39:24.737   729   755 I am_pss  : [22214,1000,com.android.settings,48032768,40583168,0]
07-13 07:39:24.753 13864 13864 I art     : Explicit concurrent mark sweep GC freed 414(18KB) AllocSpace objects, 0(0B) LOS objects, 63% free, 7MB/19MB, paused 581us total 63.858ms
07-13 07:39:24.820   729 17323 D PowerManagerService: acquireWakeLockInternal: lock=129801615, flags=0x2000000a, tag="WindowManager", ws=WorkSource{10016}, uid=1000, pid=729
07-13 07:39:24.820   729 17323 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x23
07-13 07:39:24.820   729 17323 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61612185 (in 9976 ms)
07-13 07:39:24.821   729 17323 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x23, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:24.821   729 17323 D PowerManagerService: Acquiring suspend blocker "PowerManagerService.WakeLocks".
07-13 07:39:24.870   729   756 I am_destroy_activity: [0,223991823,529,com.android.calendar/.RequestPermissionsActivity,finish-imm]
07-13 07:39:24.874 13864 13864 I am_on_stop_called: [0,com.android.calendar.RequestPermissionsActivity,destroy]
07-13 07:39:24.875   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:24.875   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:24.883   729 15804 I wm_task_removed: [529,removeAppToken: last token]
07-13 07:39:24.885   729 15804 I wm_task_removed: [529,removeTask]
07-13 07:39:24.934   729   838 D PowerManagerService: userActivityNoUpdateLocked: eventTime=61602321, event=2, flags=0x0, uid=1000
07-13 07:39:24.935   729   838 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61614321 (in 11998 ms)
07-13 07:39:24.935   729   838 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x23, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: Error getting print jobs.
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: java.util.concurrent.TimeoutException: Cannot get spooler!
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: 	at com.android.server.print.RemotePrintSpooler.bindLocked(RemotePrintSpooler.java:633)
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: 	at com.android.server.print.RemotePrintSpooler.getRemoteInstanceLazy(RemotePrintSpooler.java:602)
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: 	at com.android.server.print.RemotePrintSpooler.getPrintJobInfos(RemotePrintSpooler.java:162)
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: 	at com.android.server.print.UserState.getPrintJobInfos(UserState.java:278)
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: 	at com.android.server.print.PrintManagerService$PrintManagerImpl.getPrintJobInfos(PrintManagerService.java:160)
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: 	at android.print.IPrintManager$Stub.onTransact(IPrintManager.java:57)
07-13 07:39:24.947   729 17330 E RemotePrintSpooler: 	at android.os.Binder.execTransact(Binder.java:565)
07-13 07:39:24.953  1095  1095 I SST     : Auto time state changed
07-13 07:39:24.953 22214 22214 I sysui_count: [DateTimeSettings/auto_time|false,1]
07-13 07:39:25.075   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:25.075   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:25.275   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:25.275   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:25.475   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:25.476   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:25.676   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:25.676   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:25.876   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:25.876   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:25.942   729   838 D PowerManagerService: userActivityNoUpdateLocked: eventTime=61603328, event=1, flags=0x0, uid=1000
07-13 07:39:25.943   729   838 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61615328 (in 11996 ms)
07-13 07:39:25.943   729   838 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x23, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:26.077   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:26.077   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:26.277   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:26.277   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:26.460   729   729 W WindowManager: Attempted to remove non-existing token: android.os.Binder@5fa9715
07-13 07:39:26.478   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:26.478   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:26.679   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:26.679   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:26.682     0     0 W [3][61603.333612][07-12 20:39:25.307]migrate_irqs: 993 callbacks suppressed
07-13 07:39:26.682     0     0 W         : [3][61603.333622][07-12 20:39:25.307]IRQ32 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333630][07-12 20:39:25.307]IRQ33 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333636][07-12 20:39:25.307]IRQ34 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333645][07-12 20:39:25.307]IRQ35 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333652][07-12 20:39:25.307]IRQ36 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333659][07-12 20:39:25.307]IRQ37 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333666][07-12 20:39:25.307]IRQ38 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333673][07-12 20:39:25.307]IRQ40 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333681][07-12 20:39:25.307]IRQ41 no longer affine to CPU2
07-13 07:39:26.682     0     0 W         : [3][61603.333688][07-12 20:39:25.307]IRQ42 no longer affine to CPU2
07-13 07:39:26.879   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:26.879   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:26.965   729   838 D PowerManagerService: userActivityNoUpdateLocked: eventTime=61604350, event=2, flags=0x0, uid=1000
07-13 07:39:26.965   729   838 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61616350 (in 11996 ms)
07-13 07:39:26.965   729   838 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x23, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:26.992   729  2520 I ActivityManager: START u0 {act=android.intent.action.MAIN cmp=com.android.settings/.SubSettings (has extras)} from uid 1000 on display 0
07-13 07:39:27.003   729  2520 I wm_task_moved: [523,1,16]
07-13 07:39:27.005   729  2520 I am_create_activity: [0,148483304,523,com.android.settings/.SubSettings,android.intent.action.MAIN,NULL,NULL,0]
07-13 07:39:27.008   729  2520 I wm_task_moved: [523,1,16]
07-13 07:39:27.017   729  2520 I am_focused_activity: [0,com.android.settings/.SubSettings,startedActivity]
07-13 07:39:27.017   729  2520 D ActivityTrigger: ActivityTrigger activityPauseTrigger 
07-13 07:39:27.017   729  2520 I am_pause_activity: [0,69092980,com.android.settings/.Settings$DateTimeSettingsActivity]
07-13 07:39:27.018   729  2520 D PowerManagerService: acquireWakeLockInternal: lock=840646, flags=0x1, tag="*launch*", ws=WorkSource{1000}, uid=1000, pid=729
07-13 07:39:27.018   729  2520 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x23
07-13 07:39:27.018   729  2520 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61616350 (in 11943 ms)
07-13 07:39:27.018   729  2520 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x23, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:27.024   729  2520 D PowerManagerService: releaseWakeLockInternal: lock=129801615 [WindowManager], flags=0x0
07-13 07:39:27.024   729  2520 D PowerManagerService: userActivityNoUpdateLocked: eventTime=61604412, event=0, flags=0x1, uid=1000
07-13 07:39:27.024   729  2520 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x1
07-13 07:39:27.024   729  2520 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61616350 (in 11937 ms)
07-13 07:39:27.025   729  2520 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x1, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:27.038 22214 22214 I sysui_view_visibility: [38,0]
07-13 07:39:27.039   334  1500 E getlog  : getCurrentLogType , Line: 00489: get current log type:0 
07-13 07:39:27.040   334  1500 E getlog  : get_file_size , Line: 01819: get_file_size enter :file /data/logs/aplog/logcat_2019-7-13__4-40-35_7.log  
07-13 07:39:27.040   334  1500 E getlog  : get_file_size , Line: 01829: file /data/logs/aplog/logcat_2019-7-13__4-40-35_7.log  size:39687 KB 
07-13 07:39:27.041   334  1500 E getlog  : getDiskFreePercent , Line: 01423: TOTAL_SIZE == 5475614720 B,5347280 KB,5221 MB
07-13 07:39:27.041   334  1500 E getlog  :  
07-13 07:39:27.041   334  1500 E getlog  : getDiskFreePercent , Line: 01424: DISK_FREE == 4265115648 B,4165152 KB,4067 MB
07-13 07:39:27.041   334  1500 E getlog  :  
07-13 07:39:27.041   334  1500 E getlog  : getDiskFreePercent , Line: 01425: free_percent == 0.7789291143 
07-13 07:39:27.041   334  1500 E getlog  :  
07-13 07:39:27.042   334  1500 E getlog  : get_pid_by_file , Line: 00391: buffer is 30146
07-13 07:39:27.042   334  1500 E getlog  : ,buffer len is 6 
07-13 07:39:27.042   334  1500 E getlog  : get_pid_by_file , Line: 00401: result is "30146
07-13 07:39:27.042   334  1500 E getlog  : "
07-13 07:39:27.042   334  1500 E getlog  :  
07-13 07:39:27.042   334  1500 E getlog  : isLogProcessAlive , Line: 00301: in isLogProcessAlive, process logcat,plist->size:1
07-13 07:39:27.042   334  1500 E getlog  :  
07-13 07:39:27.042   334  1500 E getlog  : checkCmdlineByPid , Line: 00176: checkCmdlineByPid Enter 
07-13 07:39:27.042   334  1500 E getlog  : checkCmdlineByPid , Line: 00185: filename is /proc/30146/cmdline
07-13 07:39:27.042   334  1500 E getlog  :  
07-13 07:39:27.042   334  1500 E getlog  : checkCmdlineByPid , Line: 00202: readret:80,buffer is logcat -b all -v threadtime -f /data/logs/aplog/logcat_2019-7-13__4-40-35_7.log,buffer len is 79 
07-13 07:39:27.042   334  1500 E getlog  : checkCmdlineByPid , Line: 00211: buffer is logcat -b all -v threadtime -f /data/logs/aplog/logcat_2019-7-13__4-40-35_7.log,cmdline is logcat -b all -v threadtime -f /data/logs/aplog/logcat_2019-7-13__4-40-35_7.log&
07-13 07:39:27.042   334  1500 E getlog  :  
07-13 07:39:27.043   334  1500 E getlog  : checkCmdlineByPid , Line: 00243: buffer is cmdline will return 1 
07-13 07:39:27.043   334  1500 E getlog  : isLogProcessAlive , Line: 00306: in isLogProcessAlive,process logcat,pid:30146
07-13 07:39:27.043   334  1500 E getlog  :  
07-13 07:39:27.043   334  1500 E getlog  : isLogProcessAlive , Line: 00313: in isLogProcessAlive,process logcat,return result:1 
07-13 07:39:27.043   334  1500 E getlog  : mainLoop , Line: 01563: sleep 10s for next poll 
07-13 07:39:27.045 22214 22214 I am_on_paused_called: [0,com.android.settings.Settings$DateTimeSettingsActivity,handlePauseActivity]
07-13 07:39:27.057   729  1418 I am_restart_activity: [0,148483304,523,com.android.settings/.SubSettings]
07-13 07:39:27.058   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/class/power_supply/bms/current_now failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.065   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/class/power_supply/bms/resistance_now failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.065   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/class/power_supply/bms/battery_type failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.066   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/class/power_supply/bms/capacity failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.066   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/class/power_supply/bms/status failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.066   729   756 I sysui_count: [window_time_0,3]
07-13 07:39:27.067   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/class/power_supply/battery/technology failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.068   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/class/power_supply/bms/battery_type failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.076   334  1500 E getlog  : getNodeValue , Line: 01011: open /sys/devices/system/cpu/cpu3/cpufreq/cpuinfo_cur_freq failed,will return 0,error:2,err str:No such file or directory 
07-13 07:39:27.080   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:27.081   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:27.090 22214 22214 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@7ca46ec
07-13 07:39:27.090 22214 22214 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@9444ab5
07-13 07:39:27.090 22214 22214 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@dd5254a
07-13 07:39:27.090 22214 22214 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@6599cbb
07-13 07:39:27.106 22214 29494 E ActivityThread: Failed to find provider info for com.qti.smq.qualcommFeedback.provider
07-13 07:39:27.130 22214 22214 D SubSettings: Launching fragment com.android.settings.ZonePicker
07-13 07:39:27.282   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:27.282   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:27.289 22214 29304 D Index   : Indexing locale 'zh_CN_#Hans' took 16 millis
07-13 07:39:27.291 22214 22214 I am_on_resume_called: [0,com.android.settings.SubSettings,LAUNCH_ACTIVITY]
07-13 07:39:27.331 22214 29304 D Index   : Indexing locale 'zh_CN_#Hans' took 13 millis
07-13 07:39:27.482   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:27.482   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:27.508   729   778 I am_activity_launch_time: [0,148483304,com.android.settings/.SubSettings,461,3576]
07-13 07:39:27.508   729   778 I ActivityManager: Displayed com.android.settings/.SubSettings: +461ms (total +3s576ms)
07-13 07:39:27.629 22214 22235 D OpenGLRenderer: endAllActiveAnimators on 0x8aa98500 (RippleDrawable) with handle 0x8a0140a0
07-13 07:39:27.641   729  2518 D PowerManagerService: releaseWakeLockInternal: lock=840646 [*launch*], flags=0x0
07-13 07:39:27.641   729  2518 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x0
07-13 07:39:27.642   729  2518 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61616350 (in 11320 ms)
07-13 07:39:27.642   729  2518 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x0, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:27.642   729  2518 D PowerManagerService: Releasing suspend blocker "PowerManagerService.WakeLocks".
07-13 07:39:27.682   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:27.683   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:27.872   729   756 D ActivityTrigger: ActivityTrigger activityStopTrigger 
07-13 07:39:27.872   729   756 I am_stop_activity: [0,69092980,com.android.settings/.Settings$DateTimeSettingsActivity]
07-13 07:39:27.883   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 07:39:27.883   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 07:39:27.891 22214 22214 I am_on_stop_called: [0,com.android.settings.Settings$DateTimeSettingsActivity,handleStopActivity]
07-13 07:39:27.897   290   290 I sf_frame_dur: [com.android.settings/com.android.settings.Settings$DateTimeSettingsActivity,166,7,2,3,4,3,1]
07-13 07:39:27.986   729   838 D PowerManagerService: userActivityNoUpdateLocked: eventTime=61605373, event=2, flags=0x0, uid=1000
07-13 07:39:27.987   729   838 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61617373 (in 11998 ms)
07-13 07:39:27.987   729   838 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x0, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 07:39:28.074   729 17330 D AlarmManagerService: Kernel timezone updated to -600 minutes west of GMT
07-13 06:39:28.083   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 06:39:28.083   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 06:39:28.090   729 17329 I am_finish_activity: [0,148483304,523,com.android.settings/.SubSettings,app-request]
07-13 06:39:28.093   729 17329 I wm_task_moved: [523,1,16]
07-13 06:39:28.097   729 17329 I am_focused_activity: [0,com.android.settings/.Settings$DateTimeSettingsActivity,finishActivity adjustFocus]
07-13 06:39:28.097   729 17329 D ActivityTrigger: ActivityTrigger activityPauseTrigger 
07-13 06:39:28.098   729 17329 I am_pause_activity: [0,148483304,com.android.settings/.SubSettings]
07-13 06:39:28.098   729 17329 D PowerManagerService: acquireWakeLockInternal: lock=840646, flags=0x1, tag="*launch*", ws=WorkSource{1000}, uid=1000, pid=729
07-13 06:39:28.098   729 17329 D PowerManagerService: updateWakeLockSummaryLocked: mWakefulness=Awake, mWakeLockSummary=0x1 
07-13 06:39:28.098   729 17329 D PowerManagerService: updateUserActivitySummaryLocked: mWakefulness=Awake, mUserActivitySummary=0x1, nextTimeout=61617373 (in 11886 ms)
07-13 06:39:28.099   729 17329 D PowerManagerService: updateDisplayPowerStateLocked: mDisplayReady=true, policy=3, mWakefulness=1, mWakeLockSummary=0x1, mUserActivitySummary=0x1, mBootCompleted=true, mIsVrModeEnabled= false, mScreenBrightnessBoostInProgress=false
07-13 06:39:28.099   729 17329 D PowerManagerService: Acquiring suspend blocker "PowerManagerService.WakeLocks".
07-13 06:39:28.100   729   729 D ConditionProviders.SCP: onReceive android.intent.action.TIMEZONE_CHANGED
07-13 06:39:28.100   901   901 D KeyguardUpdateMonitor: received broadcast android.intent.action.TIMEZONE_CHANGED
07-13 06:39:28.109 22214 22214 I am_on_paused_called: [0,com.android.settings.SubSettings,handlePauseActivity]
07-13 06:39:28.110   729   729 D ConditionProviders.SCP: notifyCondition condition://android/schedule?days=1.2.3.4.5&start=22.0&end=7.0&exitAtAlarm=false STATE_FALSE reason=!meetsSchedule
07-13 06:39:28.110   729   729 D ConditionProviders.SCP: notifyCondition condition://android/schedule?days=3.6&start=23.30&end=10.0&exitAtAlarm=true STATE_TRUE reason=meetsSchedule
07-13 06:39:28.113   901   901 D KeyguardUpdateMonitor: handleTimeUpdate
07-13 06:39:28.116   729   744 I am_resume_activity: [0,69092980,523,com.android.settings/.Settings$DateTimeSettingsActivity]
07-13 06:39:28.124   729   729 D ConditionProviders.SCP: Scheduling evaluate for Sat Jul 13 07:00:00 GMT+10:00 2019 (1562965200000), in +20m31s900ms, now=Sat Jul 13 06:39:28 GMT+10:00 2019 (1562963968100)
07-13 06:39:28.141   729 17320 W ActivityManager: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{3121c69 13864:com.android.calendar/u0a16} (pid=13864, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
07-13 06:39:28.146 13864 29559 E AndroidRuntime: FATAL EXCEPTION: AsyncTask #1
07-13 06:39:28.146 13864 29559 E AndroidRuntime: Process: com.android.calendar, PID: 13864
07-13 06:39:28.146 13864 29559 E AndroidRuntime: java.lang.RuntimeException: An error occurred while executing doInBackground()
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.AsyncTask$3.done(AsyncTask.java:325)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:354)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.setException(FutureTask.java:223)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:242)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at java.lang.Thread.run(Thread.java:761)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: Caused by: java.lang.SecurityException: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{3121c69 13864:com.android.calendar/u0a16} (pid=13864, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1684)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1637)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.app.ActivityManagerProxy.getContentProvider(ActivityManagerNative.java:4199)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.app.ActivityThread.acquireProvider(ActivityThread.java:5478)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.app.ContextImpl$ApplicationContentResolver.acquireUnstableProvider(ContextImpl.java:2239)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.ContentResolver.acquireUnstableProvider(ContentResolver.java:1520)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.ContentResolver.query(ContentResolver.java:518)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:64)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:56)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.AsyncTaskLoader.onLoadInBackground(AsyncTaskLoader.java:312)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:69)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:66)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at android.os.AsyncTask$2.call(AsyncTask.java:305)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:237)
07-13 06:39:28.146 13864 29559 E AndroidRuntime: 	... 3 more
07-13 06:39:28.166   729 17317 I am_crash: [13864,0,com.android.calendar,814333509,java.lang.SecurityException,Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{3121c69 13864:com.android.calendar/u0a16} (pid=13864, uid=10016) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR,Parcel.java,1684]
07-13 06:39:28.177   729 17317 W ActivityManager: Force-killing crashed app com.android.calendar at watcher's request
07-13 06:39:28.186 13864 29559 I Process : Sending signal. PID: 13864 SIG: 9
07-13 06:39:28.215 22214 22214 I sysui_view_visibility: [38,100]
07-13 06:39:28.222 22214 29495 D Index   : Indexing locale 'zh_CN_#Hans' took 24 millis
07-13 06:39:28.238 22214 22214 I am_on_resume_called: [0,com.android.settings.Settings$DateTimeSettingsActivity,RESUME_ACTIVITY]
07-13 06:39:28.284   729   872 D ECPNative: ecp_payload_get(): Waiting for packet!
07-13 06:39:28.284   729   872 D ECPNative: ecp_payload_get(): Report new packet to JNI!
07-13 06:39:28.335   729 17330 D GraphicsStats: Buffer count: 13
07-13 06:39:28.337   729   919 I ActivityManager: Process com.android.calendar (pid 13864) has died
07-13 06:39:28.337   729   919 I am_proc_died: [0,13864,com.android.calendar]
```

在以上的两份log当中, 都出现了权限问题, Permission Denial: opening provider .
但是从log 里的打印的异常调用栈里, 我们并没有发现任何在 Calendar 源码里的东西. 这里只是提示了一个AsyncTaskLoader 异常.
我们先看看第二份log, 在第二份log之前, 出现了我们可以在 event log的 am_focused_activity和am_resume_activity, 知道, 
当前 Monkey正在操作的界面是Setting模块的 DateTimeSettingsActivity 界面. 我们打开Settings的这个界面, 会发现, 这个是
一个改变当前系统时间, 日期, 时区, 时间格式的界面. 那么他到操作了哪一个呢.我们不知道. 
我在这个界面, 分别操作了每一个功能, 并没有出现任何异常.

在发生错误之前, ActivityManager: Permission Denial: opening provider .这份log, 又显示, 似乎者权限有关系. 
这个和 Calendar 以及 CalendarProvider 都有关系. 
在第二份log里的进程13864, 我们只看这一个进程的log, 可以看到, 只出现了RequestPermissionsActivity 这个activity, 明显这个时请求权限的. 
那他给了权限吗. 似乎没有给. 看看随后的Systemserver进程, 也就是729号进程. 在这个进程里我们可以看到, am_destroy_activity 这个 tag, 尝试打开
AllInOneActivity 界面, 但是并没有, 而是finish .
这段 Monkey的操作, 似乎是. 打开了calendar的界面, 但是并没有给权限. 然后切换到了 Setting的界面, 改变了日期. 
### 第三份 log
于是, 我按照上面的的操作. 先打开 Calendar 的界面, 然后从 Home界面跳出来, 保证calendar 进程并没有挂掉. 
然后进入到Settings的日期界面秒, 尝试改变日期和时间. 这个时候出现了 Calendar 的错误. 
这样我们抓了另一份log.
``` 
11-27 17:17:37.200     0     0 I         : [0][  171.996633][11-27 21:17:37.053]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-27 17:17:37.200     0     0 I         : [0][  171.996649][11-27 21:17:37.053]<<-GTP-DEBUG->> [441]Touch id[ 6] release!
11-27 17:17:37.200     0     0 I         : [0][  171.996664][11-27 21:17:37.053]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-27 17:17:37.200     0     0 I         : [0][  171.996679][11-27 21:17:37.053]<<-GTP-DEBUG->> [441]Touch id[ 7] release!
11-27 17:17:37.200     0     0 I         : [0][  171.996694][11-27 21:17:37.053]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-27 17:17:37.200     0     0 I         : [0][  171.996710][11-27 21:17:37.053]<<-GTP-DEBUG->> [441]Touch id[ 8] release!
11-27 17:17:37.200     0     0 I         : [0][  171.996724][11-27 21:17:37.053]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-27 17:17:37.200     0     0 I         : [0][  171.996740][11-27 21:17:37.053]<<-GTP-DEBUG->> [441]Touch id[ 9] release!
11-15 17:17:37.229     0     0 I [0][  172.020330][11-15 21:17:37.994]<<-GTP-DEBUG->> [798]pre_touch: 01, finger:80.
11-15 17:17:37.239     0     0 I         : [0][  172.020366][11-15 21:17:37.994]<<-GTP-DEBUG->> [869]id = 0,touch_index = 0x0, pre_touch = 0x1
11-15 17:17:37.239     0     0 I         : [0][  172.020366][11-15 21:17:37.994]
11-15 17:17:37.239     0     0 I         : [0][  172.020393][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.239     0     0 I         : [0][  172.020414][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 0] release!
11-15 17:17:37.239     0     0 I         : [0][  172.020429][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.240     0     0 I         : [0][  172.020445][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 1] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020460][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020475][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 2] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020490][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020506][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 3] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020521][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020536][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 4] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020551][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020567][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 5] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020582][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020598][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 6] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020613][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020628][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 7] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020643][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020659][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 8] release!
11-15 17:17:37.241     0     0 I         : [0][  172.020673][11-15 21:17:37.994]<<-GTP-DEBUG->> [899]Devices touch Up(Slot)!
11-15 17:17:37.241     0     0 I         : [0][  172.020689][11-15 21:17:37.994]<<-GTP-DEBUG->> [441]Touch id[ 9] release!
11-15 17:17:37.241     0     0 I [0][  172.025838][11-15 21:17:37.000]utc_update_worker: Update UTC printk
11-15 17:17:37.241     0     0 I [0][  172.025867][11-15 21:17:37.000]utc_update_worker: local clock is   172.025827
11-15 17:17:37.241     0     0 I [0][  172.025885][11-15 21:17:37.000]utc_update_worker: UTC time is 2019-11-15 21:17:37(1573852657)
11-15 17:17:37.241     0     0 I [1][  172.035635][11-15 21:17:37.009]<<-GTP-DEBUG->> [798]pre_touch: 00, finger:80.
11-27 17:17:37.226   739  1189 D AlarmManagerService: Setting time of day to sec=1573852657
11-15 17:17:37.225   739  1189 W AlarmManagerService: Unable to set rtc to 1573852657: Invalid argument
11-15 17:17:37.233   739   739 D ConditionProviders.SCP: onReceive android.intent.action.TIME_SET
11-15 17:17:37.234   903   903 D KeyguardUpdateMonitor: received broadcast android.intent.action.TIME_SET
11-15 17:17:37.234   903   903 D KeyguardUpdateMonitor: handleTimeUpdate
11-15 17:17:37.258     0     0 I [3][  172.054502][11-15 21:17:37.028]<<-GTP-DEBUG->> [798]pre_touch: 00, finger:80.
11-15 17:17:37.276   739  2120 W InputMethodManagerService: Window already focused, ignoring focus gain of: com.android.internal.view.IInputMethodClient$Stub$Proxy@c226ff7 attribute=null, token = android.os.BinderProxy@978b3d2
11-15 17:17:37.293   739   765 W ActivityManager: Slow operation: 51ms so far, now at startProcess: returned from zygote!
11-15 17:17:37.293   739   765 W ActivityManager: Slow operation: 51ms so far, now at startProcess: done updating battery stats
11-15 17:17:37.293   739   765 I am_proc_start: [0,2188,10017,com.android.calendar,broadcast,com.android.calendar/.widget.CalendarAppWidgetService$CalendarFactory]
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: building log message
11-15 17:17:37.294   739   765 I ActivityManager: Start proc 2188:com.android.calendar/u0a17 for broadcast com.android.calendar/.widget.CalendarAppWidgetService$CalendarFactory
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: starting to update pids map
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: done updating pids map
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: done starting proc!
11-15 17:17:37.294   739   739 I dvm_lock_sample: [system_server,1,main,61,UserController.java,1464,BroadcastQueue.java,762,12]
11-15 17:17:37.295   739   739 D ConditionProviders.SCP: notifyCondition condition://android/schedule?days=6.7&start=23.30&end=10.0&exitAtAlarm=false STATE_FALSE reason=!meetsSchedule
11-15 17:17:37.302   739   739 D ConditionProviders.SCP: notifyCondition condition://android/schedule?days=1.2.3.4.5&start=22.0&end=7.0&exitAtAlarm=false STATE_FALSE reason=!meetsSchedule
11-15 17:17:37.303   739   739 D ConditionProviders.SCP: Scheduling evaluate for Fri Nov 15 22:00:00 AST 2019 (1573869600000), in +4h42m22s767ms, now=Fri Nov 15 17:17:37 AST 2019 (1573852657233)
11-15 17:17:37.319   739   764 I UsageStatsService: Time changed in UsageStats by -1036800 seconds
11-15 17:17:37.326   739   764 I UsageStatsService: User[0] Flushing usage stats to disk
11-15 17:17:37.368   739   880 I am_proc_bound: [0,2188,com.android.calendar]
11-15 17:17:37.407  2188  2188 I art     : Starting a blocking GC AddRemoveAppImageSpace
11-15 17:17:37.411  2188  2188 W System  : ClassLoader referenced unknown path: /system/app/Calendar/lib/arm
11-15 17:17:37.427   739   764 I UsageStatsDatabase: Time changed by -12d0h0m0s1ms. files deleted: 0 files moved: 8
11-15 17:17:37.440   295   295 I sf_frame_dur: [com.android.settings/com.android.settings.Settings$DateTimeSettingsActivity,20,2,2,0,1,0,1]
11-15 17:17:37.498   739   764 I UsageStatsService: User[0] Rollover scheduled @ 2019-11-16 14:09:55(1573927795896)
11-15 17:17:37.505  2188  2188 D ExtensionsFactory: No custom extensions.
11-15 17:17:37.560   739  2120 I am_proc_start: [0,2209,1000,com.qualcomm.timeservice,broadcast,com.qualcomm.timeservice/.TimeServiceBroadcastReceiver]
11-15 17:17:37.560   739  2120 I ActivityManager: Start proc 2209:com.qualcomm.timeservice/1000 for broadcast com.qualcomm.timeservice/.TimeServiceBroadcastReceiver
11-15 17:17:37.561   739   754 W ActivityManager: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{da498cd 2188:com.android.calendar/u0a17} (pid=2188, uid=10017) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
11-15 17:17:37.575   739   764 I am_pss  : [1117,1000,com.android.settings,42974208,35074048,0]
--------- beginning of crash
11-15 17:17:37.575  2188  2208 E AndroidRuntime: FATAL EXCEPTION: AsyncTask #1
11-15 17:17:37.575  2188  2208 E AndroidRuntime: Process: com.android.calendar, PID: 2188
11-15 17:17:37.575  2188  2208 E AndroidRuntime: java.lang.RuntimeException: An error occurred while executing doInBackground()
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.os.AsyncTask$3.done(AsyncTask.java:325)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at java.util.concurrent.FutureTask.finishCompletion(FutureTask.java:354)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at java.util.concurrent.FutureTask.setException(FutureTask.java:223)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:242)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at java.lang.Thread.run(Thread.java:761)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: Caused by: java.lang.SecurityException: Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{da498cd 2188:com.android.calendar/u0a17} (pid=2188, uid=10017) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1684)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1637)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.app.ActivityManagerProxy.getContentProvider(ActivityManagerNative.java:4199)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.app.ActivityThread.acquireProvider(ActivityThread.java:5478)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.app.ContextImpl$ApplicationContentResolver.acquireUnstableProvider(ContextImpl.java:2239)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.content.ContentResolver.acquireUnstableProvider(ContentResolver.java:1520)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.content.ContentResolver.query(ContentResolver.java:518)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:64)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.content.CursorLoader.loadInBackground(CursorLoader.java:56)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.content.AsyncTaskLoader.onLoadInBackground(AsyncTaskLoader.java:312)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:69)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.content.AsyncTaskLoader$LoadTask.doInBackground(AsyncTaskLoader.java:66)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at android.os.AsyncTask$2.call(AsyncTask.java:305)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	at java.util.concurrent.FutureTask.run(FutureTask.java:237)
11-15 17:17:37.575  2188  2208 E AndroidRuntime: 	... 3 more
11-15 17:17:37.580   739  1001 I am_crash: [2188,0,com.android.calendar,814333509,java.lang.SecurityException,Permission Denial: opening provider com.android.providers.calendar.CalendarProvider2 from ProcessRecord{da498cd 2188:com.android.calendar/u0a17} (pid=2188, uid=10017) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR,Parcel.java,1684]
11-15 17:17:37.598   739   753 I am_proc_bound: [0,2209,com.qualcomm.timeservice]
11-15 17:17:37.618   739   767 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@dc72da
11-15 17:17:37.619   739   767 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@777830b
11-15 17:17:37.624   739   767 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@f1ee2a6
11-15 17:17:37.624   739   767 V BoostFramework: BoostFramework() : mPerf = com.qualcomm.qti.Performance@b9944e7
11-15 17:17:37.627  2209  2209 W System  : ClassLoader referenced unknown path: /system/app/TimeService/lib/arm
```

在这次操作的流程我是改变了日期, 从时间戳发生的变化可以知道. 11-27变成了11-15号. 
再这次的操作过程中, 可以知道, 这个和Monkey出现的log是一致的. 基本上可以确定就是这个问题了.
### 结合代码分析
我们在看看 这次Calendar的启动, 

```
11-15 17:17:37.293   739   765 W ActivityManager: Slow operation: 51ms so far, now at startProcess: done updating battery stats
11-15 17:17:37.293   739   765 I am_proc_start: [0,2188,10017,com.android.calendar,broadcast,com.android.calendar/.widget.CalendarAppWidgetService$CalendarFactory]
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: building log message
11-15 17:17:37.294   739   765 I ActivityManager: Start proc 2188:com.android.calendar/u0a17 for broadcast com.android.calendar/.widget.CalendarAppWidgetService$CalendarFactory
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: starting to update pids map
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: done updating pids map
11-15 17:17:37.294   739   765 W ActivityManager: Slow operation: 52ms so far, now at startProcess: done starting proc!
```
从 am_proc_start 和 Start proc 可以知道, 系统是县启动的 CalendarAppWidgetService 这个类的. 而不是系统的点击activity的方式. 
同时我们可以看到时接受到了TIME_SET的广播, Calendar才启动的. 因此我们可以在 Calendar的源码搜索一下. 
我么可以知道 CalendarFactory 是一个广播接受者, 他接受了 TIMEZONE_CHANGED DATE_CHANGED TIME_SET LOCALE_CHANGED 等广播. 
这些和我们在测试的时候是基本一致的. 改变设置里的关于时间的设置, Calendar 就会挂掉. 
``` xml
<receiver android:name=".widget.CalendarAppWidgetService$CalendarFactory">
    <intent-filter>
        <action android:name="android.intent.action.TIMEZONE_CHANGED"/>
        <action android:name="android.intent.action.DATE_CHANGED"/>
        <action android:name="android.intent.action.TIME_SET"/>
        <action android:name="android.intent.action.LOCALE_CHANGED"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.PROVIDER_CHANGED"/>
        <data android:scheme="content"/>
        <data android:host="com.android.calendar"/>
    </intent-filter>
    <intent-filter>
        <action android:name="com.android.calendar.APPWIDGET_SCHEDULED_UPDATE"/>
        <data android:scheme="content"/>
        <data android:host="com.android.calendar"/>
        <data android:mimeType="vnd.android.data/update" />
    </intent-filter>
</receiver>
```

在 Calendar/src/com/android/calendar/widget/CalendarAppWidgetService.java 源码中我们可以看到
CalendarFactory 是一个 BroadcastReceiver的子类
``` java
public static class CalendarFactory extends BroadcastReceiver implements
        RemoteViewsService.RemoteViewsFactory, Loader.OnLoadCompleteListener<Cursor> {
    private static final boolean LOGD = false;

```

既然是 BroadcastReceiver 类型的, 那么我们一定要看他的 onReceive 方法

``` java
Override
public void onReceive(Context context, Intent intent) {
    if (LOGD)
        Log.d(TAG, "AppWidgetService received an intent. It was " + intent.toString());
    mContext = context;

    // We cannot do any queries from the UI thread, so push the 'selection' query
    // to a background thread.  However the implementation of the latter query
    // (cursor loading) uses CursorLoader which must be initiated from the UI thread,
    // so there is some convoluted handshaking here.
    //
    // Note that as currently implemented, this must run in a single threaded executor
    // or else the loads may be run out of order.
    //
    // TODO: Remove use of mHandler and CursorLoader, and do all the work synchronously
    // in the background thread.  All the handshaking going on here between the UI and
    // background thread with using goAsync, mHandler, and CursorLoader is confusing.
    final PendingResult result = goAsync();
    executor.submit(new Runnable() {
        @Override
        public void run() {
            // We always complete queryForSelection() even if the load task ends up being
            // canceled because of a more recent one.  Optimizing this to allow
            // canceling would require keeping track of all the PendingResults
            // (from goAsync) to abort them.  Defer this until it becomes a problem.
            final String selection = queryForSelection();

            if (mLoader == null) {
                mAppWidgetId = -1;
                mHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        initLoader(selection);
                        result.finish();
                    }
                });
            } else {
                mHandler.post(createUpdateLoaderRunnable(selection, result,
                        currentVersion.incrementAndGet()));
            }
        }
    });
}
```
在这里似乎没有什么和广播有关系的, 只要是那四个广播都会走这段代码. 
在 initLoader 方法里我们可以看到有一个log开关, 打开这个开关继续, 做上述的测试.
``` java
public void initLoader(String selection) {
    if (LOGD)
        Log.d(TAG, "Querying for widget events...");

    // Search for events from now until some time in the future
    Uri uri = createLoaderUri();
    mLoader = new CursorLoader(mContext, uri, EVENT_PROJECTION, selection, null,
            EVENT_SORT_ORDER);
    mLoader.setUpdateThrottle(WIDGET_UPDATE_THROTTLE);
    synchronized (mLock) {
        mLastSerialNum = ++mSerialNum;
    }
    mLoader.registerListener(mAppWidgetId, this);
    mLoader.startLoading();
}
```
在第二次的log里,  是已经走到了initloader1方法的. 
在这里, 看到有一个CursorLoader, 而这个恰恰是 AsyncTaskLoader 的子类. 
这就验证了上说 AsyncTaskLoader 

所以我们应该在 executor执行新的线程之前, 判断一下是否有权限, 如果没有权限的话就 直接return 掉就行了

``` java
public void onReceive(Context context, Intent intent) {
    if (LOGD)
        Log.d(TAG, "AppWidgetService received an intent. It was " + intent.toString());
    mContext = context;

    // We cannot do any queries from the UI thread, so push the 'selection' query
    // to a background thread.  However the implementation of the latter query
    // (cursor loading) uses CursorLoader which must be initiated from the UI thread,
    // so there is some convoluted handshaking here.
    //
    // Note that as currently implemented, this must run in a single threaded executor
    // or else the loads may be run out of order.
    //
    // TODO: Remove use of mHandler and CursorLoader, and do all the work synchronously
    // in the background thread.  All the handshaking going on here between the UI and
    // background thread with using goAsync, mHandler, and CursorLoader is confusing.
    int read = context.checkSelfPermission(Manifest.permission.READ_CALENDAR);
    int write = context.checkSelfPermission(Manifest.permission.WRITE_CALENDAR);

    if (read == PackageManager.PERMISSION_DENIED || write == PackageManager.PERMISSION_DENIED){
        return;
    }
    final PendingResult result = goAsync();
    executor.submit(new Runnable() {
        @Override
        public void run() {
            // We always complete queryForSelection() even if the load task ends up being
            // canceled because of a more recent one.  Optimizing this to allow
            // canceling would require keeping track of all the PendingResults
            // (from goAsync) to abort them.  Defer this until it becomes a problem.
            final String selection = queryForSelection();

            if (mLoader == null) {
                mAppWidgetId = -1;
                mHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        initLoader(selection);
                        result.finish();
                    }
                });
            } else {
                mHandler.post(createUpdateLoaderRunnable(selection, result,
                        currentVersion.incrementAndGet()));
            }
        }
    });
}
```

修改好, 继续按照上次的操作流程操作之后, 就没有出现此类的问题了. 至此, 完美的解决了问题了.

## 总结: 
对于Monkey类的问题, 要明确一下发生错误之前, Monkey 操作了些什么
    1. 操作了哪Activity的. 这点可以从 am_focused_activity 和 am_resume_activity确定 是哪一个. 这是因为Monkey主要是靠发送事件, 模拟点击的效果.
    2. 操作的界面的操作改变了什么.  比如这里的系统的时间, 时区, 日期. 而这些东西是如何影响其他的. 比如这里的是通过发送广播方式. 
   3. 进程死亡了前后, 系统哪些log, 总是伴随着发生的. 比如这里的时间, 权限. 对于和权限有关系的. 关闭权限, 打开权限的情况要试试
   4. 对于没有出现典型类型错误的 log, 我们可以看该错误类的子类. 当然子类可能在代码里很多.