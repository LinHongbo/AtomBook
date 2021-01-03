## 背景
&emsp;&emsp;之前在修改项目原生设置 Settings.apk 的问题时，曾经想修改 support-v7 包中的 RecyclerView.java 文件，但是基于Android N 的环境下，修改后的代码一直没有编入到 Settings.apk。后来从另外的一个角度对问题进行了修改，避开了修改 android-support 包的雷区。最近在学习 support-v4 包中 ViewPager.java 的触摸事件传递以及滑动冲突解决方案时，想着增加打印来增强对流程的理解，因此还是需要探究 android-support 的资源如何导入到项目工程。在此过程中遇到了一系列的问题，希望我的解决方案能够帮助到大家。
 
## 结论
1. Android N 环境使用 jack 编译方式，是不能编译出 jar 包的，这个直接导致了修改 android-support 源码后，修改后的代码没有编入项目工程。**解决方案**：我们修改对应 jar 包源码的 Android.mk 文件，临时将 jack 的编译方式关掉即可。
```
linhongbo@ubuntu:~/sdk/androidN/frameworks/support$ git diff v4/Android.mk
diff --git a/v4/Android.mk b/v4/Android.mk
index 1e8adad..248f5ab 100644
--- a/v4/Android.mk
+++ b/v4/Android.mk
@@ -264,6 +264,7 @@ support_module_src_files += $(LOCAL_SRC_FILES)

 # Here is the final static library that apps can link against.
 include $(CLEAR_VARS)
 # 关闭 jack 编译
+LOCAL_JACK_ENABLED := disabled
 LOCAL_USE_AAPT2 := true
 # 在最终生成 jar 包的配置处关闭 jack 即可
 LOCAL_MODULE := android-support-v4
 LOCAL_SDK_VERSION := 4
```

2. 修改完 Android.mk 文件后，就可以直接进行编译了，我是使用 mma 命令进行编译，此时如无意外，可以看到编译出了 javalib.jar。
此时的我们需要把该 jar 包导入到我们的项目工程中，此时分为2种情况：
* 如果我们是在 Android N 的源码环境中编译 apk，比如我们编译 Hosting 项目的定制设置 Settings.apk，那么直接进入到 Settings 的根目录进行编译即可，此时我使用的命令是 mm -B，如果能成功编译出 apk， 那么生成的修改后的 android-support 库就会合入到 Settings.apk。
* 如果是使用 Android Studio 等IDE编译第三方apk时，那么需要手动将该 jar 包导入项目工程中，并且根据实际情况，需要解决资源冲突问题  

备注：
1. 以上结论均为结果导向结论，即该文章提供了一种可行的解决方案，但对其原理的理解，有可能是错误的，各位看官需要自行判断。
2. 以上编译方式究竟是使用“mma”，“mm -B”亦或其他命令，我没有过多关注，我的需求是能编译通过即可。
3. 需要在 Android.mk 文件中哪个（或者哪些） module 关闭 jack，这个也是结果导向的，我尝试的情况是：只需要在最终生成 jar 包的 module 处关闭 jack 即可。
4. 如果 support-A.jar 需要依赖 support-B.jar，那么 support-B.jar 需要不能关闭 jack，否则 support-A.jar 会编不出来。

## 下面梳理一下我处理这件事的整个流程：

1. 之前修改 framework 的一些东西时，一般都是把一些库文件 push 到设备中，比如 Android K 的 framework.jar、framework2.jar、ext.jar，Android N 的 boot.art 等文件。修改 RecyclerView.java 时，按照常规思路，在源码环境中编译引用到 RecyclerView 的地方，应当能够自动链接到修改后的代码，但实际上是不行的（有可能有其他方案，可以让项目工程直接链接到 jack 编译的中间产物 classes.jack，不过目前没有找到这类方案）。因此当时的想法是可能产生出了一些库文件，需要 push 到设备中，但当时把所有能够想到的库文件全部 push 到设备，仍然没有达到预期效果。
2. 在 1 行不通的情况下，我对“项目工程运行时是调用设备的 android-support 还是项目的 android-support”进行了分析，如果是前者，那么 android-support 的 api 理应是通过 binder 机制进行动态链接，使用反编译工具查看 android -support.jar，发现没有任何使用 binder 的痕迹，那么说明 android-support 等库是以静态链接的方式导入到项目工程的，那么接下来就需要将源代码编译为 jar 包，然后导入到项目工程中。
3. Android N 采用 jcak 进行编译，jack 编译时，中间产物是 *.jack，不满足导入项目工程的条件（可能存在向 Android Studio 链接 jack库的方式，目前没找到这种方案），因此我们可以暂时把该 module 的 jack 编译给关闭了。此时编译出 jar 包。
4. 源码环境下链接 jar 包的情况就不阐述了，由于大多数情况下需要在源码环境编译 apk 的情况，都不会产生依赖冲突的情况（需要导入的 android-support 等代码源码也有，没有必要从外部导入，因此也不会出现版本冲突的问题），因此重点讲一下在 Android Studio 等 IDE 中导入修改后的 android-support 是如何解决依赖冲突的问题的。
5. Android Studio 有一个插件可以分析项目中是否存在依赖冲突（根据经验法则，该工具未能保证每一个依赖冲突都被检测出来，因此出现过检测时没问题，但是 build apk 还是报错：资源冲突），如图所示；如果配置了命令行工具，也可以使用命令行 gradle dependences 运行（我没配，没试过）

6. 如果出现了冲突的资源，那么只需要在导入依赖时，把部分 group 或者 module 给排除掉即可，大致配置如下
```gradle
# build.gradle
# 集中解除依赖
android {
    ...
    configurations {
       all*.exclude group: 'org.hamcrest', module: 'hamcrest-core'
    }
}

# 分别解除依赖
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:24.0.0',{
        exclude module: 'support-v4'
        exclude module: 'support-annotations'
    }
    compile 'com.android.support:design:24.0.0',{
        exclude module: 'support-v4'
        exclude module: 'support-annotations'
    }
    compile 'com.android.support:cardview-v7:24.0.0',{
        exclude module: 'support-v4'
        exclude module: 'support-annotations'
    }
    compile 'com.android.support:recyclerview-v7:24.0.0',{
        exclude module: 'support-v4'
        exclude module: 'support-annotations'
    }
    compile project(':viewPagerLibrary')
}
```

7. 如上面代码所示，我们在使用 gradle 导入依赖时，一般都会这么写“compile 'com.android.support:appcompat-v7:24.0.0'”，其中“com.android.support”就是 group，而“appcompat-v7”就是 module，“24.0.0”就是 version，上面的代码就是导入 group 为 com.android.support，module 为 appcompat-v7，版本为 24.0.0，但排除掉 module 为“support-v4”和“support-annotations”的模块。那么如何确定相对应的 group 跟 module 呢？一方面可以从 gradle dependences 的分析结果看出，另一方面在 build apk，如果报错，那么可以从冲突的类名，使用 double shift 快捷键查看当前项目存在多少个同名文件，根据右侧的文件来源可以确定哪些 module 发生了冲突。如下图，本地 support-v4.jar 与 gradle 导入的 support-annotations 发生了冲突，那么根据我的需求，我就需要把 support-annotations 给 exclude 掉。

备注：
按照以上 7 个步骤，一般就能处理掉依赖冲突问题。下面再讲一些我遇到的坑。
1. 编译出来的包在工程中存在 1 个即可，不同于使用 gradle 导入依赖，多个模块可以导入 android-support，只要保证版本一致即可。如果一个工程中导入多个编译后的 jar 包，那么有可能出现资源冲突，并且主工程的 jar 包会显示位于 android.jar 中。这个坑爹的形式导致我花了不少时间在研究 sdk 中android.jar 跟 sdk 中 android-support 是如何链接在一起的。
2. android-support-v4 24.2.0 后被拆分为 5 个部分：support-compat、support-core-utils、support-core-ui、support-media-compat、support-fragment，由于我导入的包刚好是 24.2.0 的版本，一开始跟着网上的写法“exclude module: 'support-v4'”并不能解决冲突问题，因此极度怀疑通过 exclude 这种方案依赖文件重复的可行性，这后面也花了很多时间去调试，幸好最后百度到这篇文章。因此，解决这个问题，要么把版本包从 24.2.0 降到 24.0.0或者更低版本，要么修改源码系统 Android N 的 Android.mk 文件，将其 v4 包分解为 5 个包，再按照上面的流程解决依赖冲突问题。
