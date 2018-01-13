## 什么是Android四大组件

其实可以从`AndroidManifest.xml`这个文件可以知道, 一般我们可以在`AndroidManifest.xml`看到:

```

  <manifest>
    <application>
      
      <activity>
        <intent-filter>  
        </intent-filter>
        <meta-data>
        </meta-data>
      </activity>
     
      <service>
        <intent-filter>
        </intent-filter>
        <meta-data>
        </meta-data>
      </service>
      
      <recevier>
        <intent-filter>
        </intent-filter>
        <meta-data>
        </meta-data>
      </recevier>
     
      <provider>
        <intent-filter>
        </intent-filter>
        <meta-data>
        </meta-data>
      </provider>
      
    </application>  
  </manifest>

```

上面`application`里面的四种元素`activity, service, provider, recevier`便是Android的四大组件了

## Android四大组件之Activity

activity可以理解为Android应用的对应界面, 一般学习activity，便会学习到它的生命周期, 官网给出的生命周期图如下所示:

![activity生命周期](https://box.kancloud.cn/2016-03-01_56d500de6b42c.jpg)\

当启动应用后launch某个activity时, 首先调用onCreate()方法, 该方法一般是配置数据, 变量或者做一些其他的初始化等等(比如一些购买授权之类的), 在该方法一般是设置contentView, 给各个组件添加监听器等等, 此时windows对象还未创建, 不能获取组件的尺寸, 获取的方法一般在onWindowFocusChanged(), 不过这个跟开发关系比较大, 对逆向分析可能没有什么太大的作用.调用完onCreate()方法后, 依次调用onStart(), onResume()方法, 然后activity就开始运行了, 我们将此时的activity称为当前activity.

当前activity如果被其他activity覆盖(或者跳转到其他activity,个人感觉差不多), 锁屏或者返回主屏而退到后台时(一般手机有个按钮可以选进程返回), 会调用onPause()方法, 进而调用onStop()方法, 此时activity有三种情况, 1: 系统内存不足, 强行kill掉进程, 只能重新onCreate()创建activity; 2: 重新唤醒, 此时调用onRestart()方法, 然后依次调用onStart(), onResume(), activity重新运行; 3: 直接被我们用户退出进程(比如手指划一下去掉进程), 此时调用onDestory()方法, activity结束生命周期.

除了上面图中提到的onCreate(), onStart(), onResume(), onPause(), onStop(), onDestory()这六个方法之外, 还有三个方法需要注意, 分别是onWindowFocusChanged(), onSaveInstanceState(), onRestoreInstanceState(), 这三个方法都有各自的调用时机:

onWindowFocusChanged()表示焦点改变, 就是当前activity获得焦点或者失去焦点的时候会触发, 对于获得焦点的触发时机在onResume()之后调用, 对于失去焦点的触发时机则在onPause()之后调用, 上面我们所说的打开启动应用launch activity的过程, 会在onResume()后调用一次, 在上面所描述的调用onPause()的情况之后也会调用一次.
onSaveInstanceState()表示进行状态实例相关保存, 在activity被系统销毁或者退到后台的时候会触发, 触发时机在onPause()之后, 在于onWindowFocusChanged()之后(在自己的海马玩跑了代码测试的结果), 这个方法一般是用来保存临时数据的, 比如你搞个音乐播放, 那么可以保存进度条, 时间, 音量等等这些临时数据.
onRestoreInstanceState()表示状态实例相关恢复, 在activity退回到后台之后系统不kill掉它, 然后又回到该activity调用, 或者是旋转屏幕, 键盘显示隐藏时系统销毁activity后重建时调用, 调用时机在onStart()之后, 一般是用来恢复数据显示之类的, 比如上面的音乐播放, 在onSaveInstanceState()保存临时数据后, 在onRestoreInstanceState()中可以取这些数据重新配置到新的布局组件中.

关于activity的启动模式, 是在`AndroidManifest.xml`进行配置的, 通过配置activity属性`android:launchMode`, 在ide下有代码提示会弹出四个值: standard, singleTop, singleTask, singleInstance

在说明启动模式之前, 需要明白有一个叫task的东西存在, 这个东西相当于一个栈, 栈里面存放的是activity实例, 一个应用可以有多个task(这种情况是应用有activity设置为singleInstance启动模式的情况), 或者, 不用区分task对应的应用, 只需要关注taskId来区分task也是可以的.

首先, standard是activity默认的启动模式, 不管task栈里面存不存在实例, 照样生成堆进栈里面, 比如, 我有个叫做`StandardActivity`的activity, 里面设置了个按钮, 并且设置了按钮的监听处理: 由StandardActivity跳转到StandardActivity, 然后我们启动应用, 此时未点击按钮, 只有一个task栈, 存放着一个StandardActivity实例, 点击按钮后, 尽管task栈已经有了一个StandardActivity了, 但是还是会重新创建多一个, 然后push进栈中, 此时栈有两个StandardActivity(可以用toString()方法输出进行区分).

然后是singleTop启动模式, 这个启动模式比较简单, 可以根据名字理解, 简单说, 当跳转activity时, 如果发现task栈里面有该activity实例位于栈顶, 那么直接用这个activity, 否则重新创建实例.

接着是singleTask启动模式, 这个启动模式也比较简单, 可以分为三种情况(针对设置了singleTask启动模式的activity), 1: 如果当前Activity位于栈顶, 那么直接调用此activity;  2: 如果不存在此activity, 那么创建该activity实例, 并且push进栈;   3: 如果存在activity实例, 但是不在栈顶, 那么将该activity实例之上的其他activity全部移出栈, 让该activity实例位于栈顶.

最后是singleInstance启动模式, 该模式可以有多个task栈(可以通过taskId验证), 该模式的一个用处就是不同应用之间可以共享设置了该启动模式的activity实例.

注: activity的继承关系是: context-contextWrapper->contextThemeWrapper->activity, 所以, 如果哪些方法是context所属的, 那么在activity是可以直接调用的.

## Android四大组件之Service

## Android四大组件之Recevier

## Android四大组件之Provider

