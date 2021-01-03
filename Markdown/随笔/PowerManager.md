# PowerManager

## 概述

这个类让你拥有控制设备状态的权利。使用这个api类会明显地影响设备电量的使用时长。除非你确实需要PowerManager.WakeLock,否则不要轻易使用它们，尽量使用低等级功能并确保在你不使用它们的时候立即释放PowerManager.WakeLock。

你可以通过使用Context.getSystemService来获取这个类的实例。你会使用到的最主要API就是newWakeLock()。这个方法会实例化一个PowerManager.WakeLock实例，你可以用WakeLock的方法去控制设备的电量状态。

```
PowerManager pm = (PowerManager)getSystemService(Contect.POWER_SERVICE);  
PowerManager.WakeLock wl = pm.newWakeLock(PowerManager.SCREEN_DIM_WAKE_LOCK,"MyTag");  
wl.acquire();  
// 相关逻辑操作  
wl.release();  
```

下面是该类的四个功能等级，覆盖了对系统电源的所有影响。**这些功能都是互斥的，你只能使用它们中某个**。
Flag Value | CPU | Screen |KeyBoard
---|---|---|---
PARTIAL_WAKE_LOCK | On* | Off | Off
SCREEN_DIM_WAKE_LOCK | On | Dim | Off
SCREEN_BRIGHT_WAKE_LOCK | On | Bright | Off
FULL_WAKE_LOCK | On | Bright | Bright
* 如果你使用的是PARTIAL_WAKE_LOCK的锁，CPU会一直保持运行状态，哪怕屏幕展示时间超时甚至用户手动点击电源按钮锁屏也无法改变CPU运行状态。而在其他三个状态的锁下，CPU虽然会运行，但当用户使用电源按钮时，CPU还是会陷入休眠状态。

另外，你可以增加两个只会影响屏幕状态的标志。**并且当锁的级别设置为PARTIAL_WAKE_LOCK时，这些标志不生效**。
Flag Value | Description
---|---
ACQUIRE_CAUSES_WAKEUP | 正常情况下，锁并不会打开屏幕照明功能，事实上，它们只是在屏幕被点亮（比如在Activity中）时让它**保持**照明功能，但如果你使用这个标签，当你获得锁时就会立刻将屏幕和键盘状态打开。一个典型的用法就是在需要用户立马看到屏幕和键盘的通知时使用这个标签，比如信息通知，来电等。
ON_AFTER_RELEASE | 如果这个标签被使用，activity计时器会在锁被释放的时候重置，使屏幕照明时间延长。这个标签可以用来当你在各种锁状态切换过程中减少闪烁。

任何使用WakeLock的程序都必须在应用的mainfiest中声明Android.permission.WAKE_LOCK权限！

## 常量
```java
// 保证CPU处于运行状态，但允许屏幕和键盘处于背光熄灭状态。当用户按下电源键，屏幕熄灭，但是CPU将保持运行直到所有PARTIAL_WAKE_LOCK被释放。
public static final int PARTIAL_WAKE_LOCK = 0x00000001;
// 源码未注释
public static final int SUSPEND_WAKE_LOCK = 0x00001000;
// 确保屏幕处于点亮状态（可以是暗光），但允许键盘背光处于熄灭状态。当用户按下电源键，该锁会被系统暗中释放掉，造成CPU和屏幕都会处于休眠状态，与PARTIAL_WAKE_LOCK相反。
// 大多数apk应该使用android.view.WindowManager.LayoutParams#FLAG_KEEP_SCREEN_ON来代替这个锁。当用户在不同apk之间进行切换时，系统可以更合理地进行管理，并且不需要特殊权限。
@Deprecated
public static final int SCREEN_DIM_WAKE_LOCK = 0x00000006;
// 大致同SCREEN_DIM_WAKE_LOCK。
@Deprecated
public static final int SCREEN_BRIGHT_WAKE_LOCK = 0x0000000a;
// 确保屏幕和键盘背光都处于最大亮度，其余SCREEN_DIM_WAKE_LOCK。
@Deprecated
public static final int = FULL_WAKE_LOCK = 0x0000001a;
// 当距离传感器激活时熄灭屏幕。如果距离传感器感应到物体在接近，屏幕会被立即熄灭；当物体离开时，屏幕会被再次点亮。
// 和FULL_WAKE_LOCK、SCREEN_BRIGHT_WAKE_LOCK、SCREEN_DIM_LOCK不同，该锁不能阻止设备进入休眠状态。通常情况下，如果没有用户激活并且没有持有其他锁，设备会进入休眠状态。但是当屏幕被距离传感器熄灭时，设备并不会进入休眠状态，因为它等效于一个持续的用户操作。
// 并不是所有的设备都拥有距离传感器，使用isWakeLockLevelSupported可以确定设备是否支持距离传感器。
// 不能在ACQUIRE_CAUSES_WAKEUP下使用。
@hide
public static final int PROXIMITY_SCREEN_OFF_WAKE_LOCK = 0x00000020;
// 类似子网掩码，用来跟一些标识作位操作。
@hide
public static final int WAKE_LOCK_LEVEL_MASK = 0x0000ffff;
// 当获取锁时将屏幕亮度打开。
// 正常情况下锁并不会唤醒设备，它们只是让已经开启的屏幕状态保持唤醒状态。就像视频播放应用软件正常情况下位播放视频，只有当弹出通知或设备需要唤醒才属于例外情况，用这个标签就是这个用法。
// 这个常量不能和PARTIAL_WAKE_LOCK一同使用
public static final int ACQUIRE_CAUSES_WAKEUP = 0x10000000;
// 当锁被释放时，activity计时器会被重置，可以使屏幕保持一小段时间的点亮状态。
public static final int ON_AFTER_RELEASE = 0x20000000;
// mWakeLock.release(int)的标志位，用来延迟释放PROXIMITY_SCREEN_OFF_WAKE_LOCK,直到距离传感器返回一个负数。
@hide
public static final int WAIT_FOR_PROXIMITY_NEGATIVE = 1;
// 最大亮度
@hide
public static final int BRIGHTNESS_ON = 255;
// 最小亮度
@hide
public static final int BRIGHTNESS_OFF = 0;

// 注意：如果需要添加或者修改用户操作事件常量，请必须更新android.os.BatteryStats和PowerManager.h。
@hide
public static final int USER_ACTIVITY_EVENT_OTHER = 0;
@hide
public static final int USER_ACTIVITY_EVENT_BUTTON = 1;
@hide
public static final int USER_ACTIVITY_EVENT_TOUCH = 2;
@hide
public static final int USER_ACTIVITY_FLAG_NO_CHANGE_LIGHTS = 1<<0;
@hide
public static final int GO_TO_SLEEP_REASON_USER = 0;
@hide
public static final int GO_TO_SLEEP_REASON_DEVICE_ADMIN = 1;
@hide
public static final int GO_TO_SLEEP_REASON_TIMEOUT = 2;
```

## 方法
```java
// android.Manifest.permission#DEVICE_POWER
// levelAndFlags:PARTIAL_WAKE_LOCK、SCREEN_DIM_WAKE_LOCK、SCREEN_BRIGHT_WAKE_LOCK、FULL_WAKE_LOCK（单选）；ACQUIRE_CAUSES_WAKEUP、ON_AFTER_RELEASE（多选）
// 创建该锁不需要权限，但是mWakeLock.acquire()、mWakeLock.release()则需要申请权限；如果需要保持屏幕常量，应当使用android.view.WindowManager.LayoutParams#FLAG_KEEP_SCREEN_ON来代替这些标签。
public WakeLock newWakeLock(int levelAndFlags, String tag){
    // 校验参数是否正确，否则抛出异常。
    validateWakeLockParameters(levelAndFlags, tag);
    return new WakeLock(levelAndFlags, tag, mContext.getOpPackageName());
}

// 当有用户操作时调用该方法，这个方法会重置灭屏倒计时，并使屏幕处于亮屏状态（当设备不处于休眠状态），当设备处于休眠状态时，该方法不会唤醒设备。
// 需要权限android.Manifest.permission#DEVICE_POWER
public void userActivity(long when, boolean noChangeLights);

// 强制设备进入休眠状态
// 需要权限android.Manifest.permission#DEVICE_POWER
public void goToSleep(long time);

// 唤醒设备
// 需要权限android.Manifest.permission#DEVICE_POWER
public void wakeUp(long time);

// 使设备进入类休眠状态（比如屏幕变暗）。当设备是唤醒状态，则会进入类休眠状态，否则不产生效果。当类休眠结束后是唤醒状态还是休眠状态取决于这段时间有没有用户操作。
@hide
public void nap(long time);

// 设置背光亮度（屏幕、键盘，按键）
// 取值范围：0-255
@hide
public void setBacklightBrightness(int brightness);

// 判断当前设备是否支持某种级别的锁
@hide
public boolean isWakeLockLevelSupported(int level);

// 判断当前设备屏幕是否处于亮屏状态（bright、dim）
public boolean isScreenOn();

// 重启设备
public void reboot(String reason);
```
