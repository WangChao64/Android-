# Activity 生命周期浅析
 
 Activity作为四大组件之一，出现的频率相当高，基本上我们在android的各个地方都能看见它的踪影，因此深入了解Activity，对于开发高质量应用程序是很有帮助的。今天我们就来详细地聊聊Activity的生命周期，以便我们在以后的开发中能如鱼得水。

一、初识Activity
  在日常应用中Activity是与用户交互的接口，它提供了一个用户完成相关操作的窗口。当我们在开发中创建Activity后，通过调用setContentView(View)方法来给该Activity指定一个布局界面，而这个界面就是提供给用户交互的接口。Android系统中是通过Activity栈的方式来管理Activity的，而Activity自身则是通过生命周期的方法来管理的自己的创建与销毁，既然如此，现在我们就来看看Activity生命周期是如何运作的。

二、Activity 的形态
Active/Running: 
Activity处于活动状态，此时Activity处于栈顶，是可见状态，可与用户进行交互。 
Paused： 
当Activity失去焦点时，或被一个新的非全屏的Activity，或被一个透明的Activity放置在栈顶时，Activity就转化为Paused状态。但我们需要明白，此时Activity只是失去了与用户交互的能力，其所有的状态信息及其成员变量都还存在，只有在系统内存紧张的情况下，才有可能被系统回收掉。 
Stopped： 
当一个Activity被另一个Activity完全覆盖时，被覆盖的Activity就会进入Stopped状态，此时它不再可见，但是跟Paused状态一样保持着其所有状态信息及其成员变量。 
Killed： 
当Activity被系统回收掉时，Activity就处于Killed状态。 
Activity会在以上四种形态中相互切换，至于如何切换，这因用户的操作不同而异。了解了Activity的4种形态后，我们就来聊聊Activity的生命周期。

三、Activity 生命周期一览
这里我们先来看看这一张经典的生命周期流程图： 
 
  相信大部分人对这种流程图并不陌生，嗯，我们下面主要聊得话题就是围绕这张流程图了。我们先有个大概印象，后面我们分析完后再回来看，就相当清晰了。

四、典型的生命周期
  所谓的典型的生命周期就是在有用户参与的情况下，Activity经历从创建，运行，停止，销毁等正常的生命周期过程。我们这里先来介绍一下几个主要方法的调用时机，然后再通过代码层面来验证其调用流程。 
onCreate : 该方法是在Activity被创建时回调，它是生命周期第一个调用的方法，我们在创建Activity时一般都需要重写该方法，然后在该方法中做一些初始化的操作，如通过setContentView设置界面布局的资源，初始化所需要的组件信息等。 
onStart : 此方法被回调时表示Activity正在启动，此时Activity已处于可见状态，只是还没有在前台显示，因此无法与用户进行交互。可以简单理解为Activity已显示而我们无法看见摆了。 
onResume : 当此方法回调时，则说明Activity已在前台可见，可与用户交互了（处于前面所说的Active/Running形态），onResume方法与onStart的相同点是两者都表示Activity可见，只不过onStart回调时Activity还是后台无法与用户交互，而onResume则已显示在前台，可与用户交互。当然从流程图，我们也可以看出当Activity停止后（onPause方法和onStop方法被调用），重新回到前台时也会调用onResume方法，因此我们也可以在onResume方法中初始化一些资源，比如重新初始化在onPause或者onStop方法中释放的资源。 
onPause : 此方法被回调时则表示Activity正在停止（Paused形态），一般情况下onStop方法会紧接着被回调。但通过流程图我们还可以看到一种情况是onPause方法执行后直接执行了onResume方法，这属于比较极端的现象了，这可能是用户操作使当前Activity退居后台后又迅速地再回到到当前的Activity，此时onResume方法就会被回调。当然，在onPause方法中我们可以做一些数据存储或者动画停止或者资源回收的操作，但是不能太耗时，因为这可能会影响到新的Activity的显示——onPause方法执行完成后，新Activity的onResume方法才会被执行。 
onStop : 一般在onPause方法执行完成直接执行，表示Activity即将停止或者完全被覆盖（Stopped形态），此时Activity不可见，仅在后台运行。同样地，在onStop方法可以做一些资源释放的操作（不能太耗时）。 
onRestart :表示Activity正在重新启动，当Activity由不可见变为可见状态时，该方法被回调。这种情况一般是用户打开了一个新的Activity时，当前的Activity就会被暂停（onPause和onStop被执行了），接着又回到当前Activity页面时，onRestart方法就会被回调。 
onDestroy :此时Activity正在被销毁，也是生命周期最后一个执行的方法，一般我们可以在此方法中做一些回收工作和最终的资源释放。 
下面我们通过程序来验证上面流程中的几种比较重要的情况，同时观察生命周期方法的回调时机。

五、验证几个主要的生命周期情况
我们案例代码如下：

package com.cmcm.activitylifecycle;
 
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
 
public class MainActivity extends AppCompatActivity {
 
    Button bt;
 
    /**
     * Activity创建时被调用
     * @param savedInstanceState
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        LogUtils.e("onCreate is invoke!!!");
        bt= (Button) findViewById(R.id.bt);
        bt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent i = new Intent(MainActivity.this,SecondActivity.class);
                startActivity(i);
            }
        });
    }
 
 
    /**
     * Activity从后台重新回到前台时被调用
     */
    @Override
    protected void onRestart() {
        super.onRestart();
        LogUtils.e("onRestart is invoke!!!");
    }
 
    /**
     *Activity创建或者从后台重新回到前台时被调用
     */
    @Override
    protected void onStart() {
        super.onStart();
        LogUtils.e("onStart is invoke!!!");
    }
 
 
    /**
     *Activity创建或者从被覆盖、后台重新回到前台时被调用
     */
    @Override
    protected void onResume() {
        super.onResume();
        LogUtils.e("onResume is invoke!!!");
    }
 
    /**
     *  Activity被覆盖到下面或者锁屏时被调用
      */
    @Override
    protected void onPause() {
        super.onPause();
        LogUtils.e("onPause is invoke!!!");
    }
 
    /**
     *退出当前Activity或者跳转到新Activity时被调用
     */
    @Override
    protected void onStop() {
        super.onStop();
        LogUtils.e("onStop is invoke!!!");
    }
 
    /**
     *退出当前Activity时被调用,调用之后Activity就结束了
     */
    @Override
    protected void onDestroy() {
        super.onDestroy();
        LogUtils.e("onDestroy is invoke!!!");
    }
}
下面我们俩综合分析几种生命周期方法的调用情况
1.我们先来分析Activity启动过程中所调用的生命周期方法，运行程序截图如下： 
Snip20160716_2 
从Log中我们可以看出Activity启动后，先调用了onCreate方法，然后是onStart方法，最后是onResume方法，进入运行状态，此时Activity已在前台显示。因此， 
Activity启动–>onCreate()–>onStart()–>onResume()依次被调用

2.当前Activity创建完成后，按Home键回到主屏。按如上操作运行截图： 
Snip20160716_4 
我们在Activity创建完成后，点击Home回调主界面时，可以发现此时onPause方法和onStop方法被执行，也就是点击Home键回到主界面(Activity不可见)–>onPause()–>onStop()依次被调用

3.当我们点击Home键回到主界面后，再次点击App回到Activity时，调用结果如下：Snip20160716_5 
我们可以发现重新回到Activity时，调用了onRestart方法，onStart方法，onResume方法。因此， 
当我们再次回到原Activity时–>onRestart()–>onStart()–>onResume()依次被调用

4.当我们在原有的Activity的基础上打新的Activity时，调用结果如下： 
Snip20160716_6 
我们可看到原来的Activity调用的了onPause方法和onStop方法。也就是说 
在原Activity的基础上开启新的Activity，原Activity生命周期执行方法顺序为–>onPause()–>onStop()，事实上跟点击home键是一样的。但是这里有点要注意的是如果新的Activity使用了透明主题，那么当前Activity不会回调onStop方法。同时我们发现新Activity（SecondActivity）生命周期方法是在原Activity的onPause方法执行完成后才可以被回调，这也就是前面我们为什么说在onPause方法不能操作耗时任务的原因了。假设：在一个activity启动模式为singleTop且本身在前台时,再次启动当前activity的生命周期如下:
onPause()->onNewIntent->onResume

5 当我们点击Back键回退时，回调结果如下： 
Snip20160716_7
从Log我们可以看出，当点击Back键回退时，相当于退出了当前Activity，Activity将被销毁，因此 
退出当前Activity时–>onPause()–>onStop()–>onDestroy()依次被调用。假设：先启动Activity A->在Activity A中启动Activity B->按Back键
按Back键时的生命周期为B.onPause()->A.onRestart->A.onStart->A.onResume-> B.onStop->B.onDestroy

小结：到这里我们来个小结，当Activity启动时，依次会调用onCreate(),onStart(),onResume()，而当Activity退居后台时（不可见，点击Home或者被新的Activity完全覆盖），onPause()和onStop()会依次被调用。当Activity重新回到前台（从桌面回到原Activity或者被覆盖后又回到原Activity）时，onRestart()，onStart()，onResume()会依次被调用。当Activity退出销毁时（点击back键），onPause()，onStop()，onDestroy()会依次被调用，到此Activity的整个生命周期方法回调完成。现在我们再回头看看之前的流程图，应该是相当清晰了吧。嗯，这就是Activity整个典型的生命周期过程。下篇我们再来聊聊Activity的异常生命周期。 

补充：灭屏时,onPause()和onStop()会依次被调用
屏亮时,onRestart()，onStart()，onResume()会依次被调用。


Activity异常生命周期
  异常的生命周期是指Activity被系统回收或者当前设备的Configuration发生变化（一般指横竖屏切换）从而导致Activity被销毁重建。异常的生命周期主要分以下两种情况：

1、相关的系统配置发生改变导致Activity被杀死并重新创建（一般指横竖屏切换）
2、内存不足导致低优先级的Activity被杀死

1、相关的系统配置发生改变导致Activity被杀死并重新创建（一般指横竖屏切换）
我们先来分析第一种情况，当Activity处于竖屏状态，如果突然旋转屏幕，由于系统配置发生了变化，在默认的情况下，Activity会被销毁并重新创建，当然我们可以人为干预来防止这种情况。在这里首先我们使用上一篇文章的测试代码来验证此过程，代码如下：

package com.cmcm.activitylifecycle;
 
import android.content.Intent;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
 
public class MainActivity extends AppCompatActivity {
 
    Button bt;
 
    /**
     * Activity创建时被调用
     * @param savedInstanceState
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        LogUtils.e("onCreate is invoke!!!");
        bt= (Button) findViewById(R.id.bt);
        bt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent i = new Intent(MainActivity.this,SecondActivity.class);
                startActivity(i);
            }
        });
    }
 
    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        LogUtils.e("onSaveInstanceState is invoke!!!");
    }
 
    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        LogUtils.e("onRestoreInstanceState is invoke!!!");
    }
 
    /**
     * Activity从后台重新回到前台时被调用
     */
    @Override
    protected void onRestart() {
        super.onRestart();
        LogUtils.e("onRestart is invoke!!!");
    }
 
    /**
     *Activity创建或者从后台重新回到前台时被调用
     */
    @Override
    protected void onStart() {
        super.onStart();
        LogUtils.e("onStart is invoke!!!");
    }
 
 
    /**
     *Activity创建或者从被覆盖、后台重新回到前台时被调用
     */
    @Override
    protected void onResume() {
        super.onResume();
        LogUtils.e("onResume is invoke!!!");
    }
 
    /**
     *  Activity被覆盖到下面或者锁屏时被调用
      */
    @Override
    protected void onPause() {
        super.onPause();
        LogUtils.e("onPause is invoke!!!");
    }
 
    /**
     *退出当前Activity或者跳转到新Activity时被调用
     */
    @Override
    protected void onStop() {
        super.onStop();
        LogUtils.e("onStop is invoke!!!");
    }
 
    /**
     *退出当前Activity时被调用,调用之后Activity就结束了
     */
    @Override
    protected void onDestroy() {
        super.onDestroy();
        LogUtils.e("onDestroy is invoke!!!");
    }
}

  代码比较简单，我们重写了onSaveInstanceState方法和onRestoreInstanceState方法，这两个方法后面我们会详细介绍，这里先看看运行结果： 
Snip20160716_8
  从Log中我们可以看出当我们正常启动Activity时，onCreate，onStart，onResume方法都会依次被回调，而如果我们此时把竖屏的Activity人为的调整为横屏，我们可以发现onPause，onSaveInstanceState，onStop，onDestroy，onCreate，onStart，onRestoreInstanceState，onResume依次被调用，单从调用的方法我们就可以知道，Activity先被销毁后再重新创建，其异常生命周期如下： 
Snip20160716_9 
  现在异常生命周期的流程我们大概也就都明白，但是onSaveInstanceState和onRestoreInstanceState方法是干什么用的呢？实际上这两个方法是系统自动调用的，当系统配置发生变化后，Activity会被销毁，也就是onPause，onStop，onDestroy会被依次调用，同时因为Activity是在异常情况下销毁的，android系统会自动调用onSaveInstanceState方法来保存当前Activity的状态信息，因此我们可以在onSaveInstanceState方法中存储一些数据以便Activity重建之后可以恢复这些数据，当然这个方法的调用时机必须在onStop方法之前，也就是Activity停止之前。至跟onPause方法的调用时机可以随意。而通过前面的Log信息我们也可以知道当Activity被重新创建之后，系统还会去调用onRestoreInstanceState方法，并把Activity销毁时通过onSaveInstanceState方法保存的Bundle对象作为参数同时传递给onRestoreInstanceState和onCreate方法，因此我们可以通过onRestoreInstanceState和onCreate方法来判断Activity是否被重新创建，倘若被重建了，我们就可以对之前的数据进行恢复。从Log信息，我们可以看出onRestoreInstanceState方法的调用时机是在onStart之后的。这里有点需要特别注意，onSaveInstanceState和onRestoreInstanceState只有在Activity异常终止时才会被调用的，正常情况是不会调用这两个方法的。 
  到这里大家可能还有一个疑问，onRestoreInstanceState和onCreate方法都可以进行数据恢复，那到底用哪个啊?其实两者都可以，两者的区别在于，onRestoreInstanceState方法一旦被系统回调，其参数Bundle一定不为空，无需额外的判断，而onCreate的Bundle却不一定有值，因为如果Activity是正常启动的话，Bundle参数是不会有值的，因此我们需要额外的判断条件，当然虽说两者都可以数据恢复，但更倾向于onRestoreInstanceState方法。 
  最后还有点我们要知道的是，在onSaveInstanceState方法和onRestoreInstanceState方法中，android系统会自动帮我们恢复一定的数据，如当前Activity的视图结构，文本框的数据，ListView的滚动位置等，这些View相关的状态系统都会帮我们恢复，这是因为每个View也有onSaveInstanceState方法和onRestoreInstanceState方法，我们来测试一个例子，在EditText文本框中输入数据然后切换横竖屏幕，结果如下： 
Snip20160716_11Snip20160716_10 
由此可以知道，系统确实帮我们恢复了Activity结构和View的相关数据。

2、内存不足导致低优先级的Activity被杀死
  接下来我们来聊聊内存不足导致低优先级的Activity被杀死，然后重建，其实也不用聊了，其数据的存储过程和恢复过程跟上面的情况基本没差。我们还是继续聊聊Activity被杀死的情况吧，当系统内存不足的时候，系统就会按照一定的优先级去杀死目标Acitivity的进程来回收内存，并且此时Activity的onSaveInstanceState方法会被调用来存储数据，并在后续Activity恢复时调用onRestoreInstanceState方法来恢复数据，所以为了Activity所在进程尽量不被杀死，我们应该尽量让其保持高的优先级。那什么样的优先级较高呢？为了更深入了解这个问题，我们先来进入一个新的话题，Android 进程层次。

Android 进程层次
  在android系统中最重要的进程被称为前台进程，然后依次是任何可见进程、服务进程、后台进程，最后是空进程。接下来我们将进一步展开。 
  在开始之前我们先要明确一个问题，当我们谈论进程优先级的时候是以 activity、service 这样的组件来说的，但请注意这些优先级是在进程的级别上的，而非组件级别。只要有一个组件是前台进程，就会将整个进程变为前台进程。同时我们要知道绝大多数应用是单进程的，如果我们有生命周期差异很大的不同部分或者某个部分非常重量型，那么我们强烈建议把它们分为不同的进程，以便让重量级进程尽早被系统回收。 
明白这点后我们再看看下面一张图（图来自Who lives and who dies? Process priorities on Android）： 
android_process
从图中我们可以看出共有5中优先级线程，Foreground Processes ，Visible Processes ，Service Processes ， Background Processes ， Empty Processes ；

1.Foreground Processes(前台进程)
  系统中前台进程的数量很少(这点从图中也是可以看出来的), 前台进程几乎不会被杀死. 只有当内存低到无法保证所有的前台进程同时运行时才会选择杀死某个前台进程.以下几种都属于前台进程： 
a. 处于前台正与用户交互的activity 
b. 与前台activity绑定的service 
c. 调用了startForeground()方法的service 
d. 正在执行onCreate(), onStart(), 或onDestroy()方法的service 
e. 正在执行onReceive()方法的BroadcastReceiver. 
凡是包含以上任意一种情况的进程都是前台进程。当然一些 activity 在依靠他们成为前台进程的同事，也可能依赖 bound service 。任何进程，如果它持有一个绑定到前台 activity 的服务，那么它也被赋予了同样的前台优先级。

2.Visible Processes(可视进程)
  此时如果一个Activity可见但并非处于前台时，如在Activity中弹出了一个对话框，从而导致Activity可见但位于后台无法与用户交互，这个进程就可以被视为可见进程，同时我们也必须明白可见 activity 的 bound service 和 content provider 也处于可见进程状态。这同样是为了保证使用中的 activity 所依赖的进程不会被过早地杀掉。但还是需要注意的是，只是可见并不意味着不能被杀掉。如果来自前台进程的内存压力过大，可见进程仍有可能被杀掉。从用户的角度看，这意味着当前 activity 背后的可见 activity 会被黑屏代替。当然，倘若我们正确地重建 activity ，在前台 activity 关闭之后，我们的进程和 activity 会立刻恢复而没有数据损失。

3.Service Processes(服务进程)
  如果我们通过startService()启动一个service服务，那么它被看作是一个服务进程。对于许多在后台做处理（如异步加载数据，获取耗时资源等）而没有立即成为前台服务的app都属于这种情况。这是比较常见也是最合理的后台处理方式，这种进程只有在可见进程和前台进程内存不足时才有可能被杀掉。

4.Background Processes(后台进程)
  假如我们的Activity目前是前台进程，但是这时候，我们点Home键，将导致onPause，onStop方法被调用，我们的进程也就变成了后台进程，当然我们的后台进程并不会被立马杀死，所以这些进程会保留一段时间，直到更高优先级进程需要内存的时候才被回收，并且是按照最近最少使用顺序（最少使用的会被优先回收）。很多时候我们会发现手机的内存大都是被后台App进程占用了大部分空间，而android系统这样做的好处是可以使用我们在下次重新打开此进程的app时无需重新分配和加载资源，从而拥有更好的用户体验。 
。然而内存不足的时候，他们仍然会被杀掉，所以我们应该和可见 activity 处理情况一样，应该尽量能够在不丢失用户状态的情况下重建这些 activity ，以便达到更佳的用户体验。

5.Empty Processes(空进程)
  在任何层次中，空进程都是最低优先级的。如果我们的进程不属于以上类别，那它就是空进程。空进程是没有活跃的组件，只是出于缓存的目的而被保留（为了更加有效地使用内存而不是完全释放掉），只要 Android 系统内存需要可以随时杀掉它们。

嗯，现在我们应该对android进程有个比较明确的概念了，回到之前的的Activity内存不足被android系统杀死的话题，为了防止一些重要的Activity不被意外杀死，我们应该让当前所在进程的Activity保持较高的优先级，如使其变为前台进程，或者通过service去绑定，也可以让其成为单独的进程。当然如果真的不幸被杀死，我们也应该尽量通过onSaveInstanceState方法和onRestoreInstanceState方法来保存和恢复数据，以便获得更佳的用户体验。

解决Activity销毁重建问题
通过上面的分析我们知道当系统配置发生变化后，Activity会被重建，那有没有办法使其不重建呢？方法自然是有的，那就是我们可以给Activity指定configChange属性，当我们不想Activity在屏幕旋转后导致销毁重建时，可以设置configChange=“orientation”；当SDK版本大于13时，我们还需额外添加一个“screenSize”的值，对于这两个值含义如下： 
orientation：屏幕方向发生变化，配置该参数可以解决横竖屏切换时，Activity重建问题（API<13） 
screenSize：当设备旋转时，屏幕尺寸发生变化，API>13后必须配置该参数才可以保证横竖切换不会导致Activity重建。 
说白了就是设置了这两个参数后，当横竖屏切换时，Activity不会再重建并且也不会调用之前相关的方法，取而代之的是回调onConfigurationChanged方法。案例代码如下：

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.cmcm.activitylifecycle" >
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme" >
        <activity android:name=".MainActivity"
            android:configChanges="orientation|screenSize"
            >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
 
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
 
        <activity android:name=".SecondActivity"></activity>
    </application>
</manifest>

在MainActivity中我们重写onConfigurationChanged，原型如下：

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        LogUtils.e("onConfigurationChanged is invoke!!!");
    }

我们运行程序，然后执行横竖屏切换，看看打印的Log信息： 
 
从Log可以看到Activity确实没有重建，并且也没有回调onSaveIntanceState方法和RestoreInstanceState方法来存储或者恢复数据，相反的是onConfigurationChanged方法被回调了，因此我们在这种情况下，可以在此方法中做一些额外的工作。
