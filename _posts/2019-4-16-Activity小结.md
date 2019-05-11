---
title: Android_Activity小结
date: 2019-4-16 12:00:00
categories:
- Android/基础
tags: Android
---

该系列主要是记录、回顾之前的学习和一些笔记。
转载请注明！
  
---
Activity在应用中的表现为界面，它会加载指定的布局文件来显示各种UI元素，同时，用户可以和这些UI元素进行交互。App便是由一个或多个Activity组成。  

---
#### Activity生命周期
- Activity的生命周期示意图

  ![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1551873034633&di=17f69ed190d2a0328e2d5a0fa302615d&imgtype=0&src=http%3A%2F%2Fs15.sinaimg.cn%2Fbmiddle%2F0065FsHCty6VWNZBEg60e%26690)
  
- Activity生命周期包含最主要的7个生命周期函数，分别是onCreate(),onStart(),onResume(),onPause(),onStop(),onDestroy(),onRestart()。
    - onCreate():在Activity第一次被创建的时候调用，并且在该方法中进行Activity的初始化操作，如加载布局、绑定事件。  
    - onStart():该方法在Activity由不可见变为可见的时候调用。在Activity的onCreate()函数调用之后被调用，此时的Activity还处在不可见状态，当Activity马上可见时，onStart() 就会被调用。 
    - onResume():该方法在Activity变为可见状态，准备好和用户进行交互的时候调用，此时的Activity一定位于返回栈的栈顶，并且处于运行状态。并且在执行完该方法后，Activity就会请求AMS([AMS介绍](https://www.cnblogs.com/neo-java/p/7146424.html))渲染它所管理的视图。  
    - onPause():该方法在系统准备启动或恢复另一个Activity的时候调用。也就是当Activity的状态即将由可见状态变为不可见状态时调用。  
    - onStop():该方法在Activity完全不可见的时候调用。该方法与onPause()方法不同点在于，当新启动是一个对话框的Activity，那么onPause()会执行，而onStop()不会。
    - onDestroy():该方法会在Activity被销毁之前调用，之后的Activity状态将变为销毁状态。一般Activity的内存释放也会在该时期进行。  
    - onRestar():该方法会在Activity由Stop状态变为Start状态之前调用。当Activity由完全不见重新可以和用户交互的时候会调用。
    
- 在Activity生命周期中，还有额外的两个回调方法：onSaveInstanceState(),onRestoreInstanceState()。这两个方法会在特定的时期被触发。
    - onSaveInstanceState():当Activity有可能被销毁，但是还没有被销毁时，该Activity的onSaveInstanceState函数就会被执行，并且该方法会保存当前Activity的状态。除非该Activity是被用户主动销毁的，如当用户按back键时。例如：
        - 用户从当前Activity切换到桌面；
        - 长按Home键，选择运行其他的程序时；
        - 按下电源键（关闭屏幕显示）时；
        - 从Activity A启动一个新的Activity时；
        - 屏幕方向切换时，如从横屏切换到竖屏；
        - 电话打入等情况发生时；
        - 总结一句话，一切让当前的Activity彻底在屏幕上看不见，但是没有主动销毁Activity的操作都会触发该方法。
    - onRestoreInstanceState():当Activity确实被系统重新回收，重新创建时才会调用。该方法可以取出onSaveInstanceState()方法保存的数据，并用这些数据恢复之前Activity的状态。值得注意的一点是，在onSaveInstanceState()方法中保存的数据，我们可以在onCreate()和onSaveInstanceState()方法中获取到，这两个方法的区别在于，onSaveInstanceState()方法一旦被调用，就表明onSaveInstanceState()方法的Bundle参数一定是有值的，不需要做额外的null判断。而onCreate()方法需要。
    
- 我们可以将Activity的完整的生命周期分为三个部分。  
  - 完整生存期(Activity在onCreate()方法和onDestroy()方法之间所经历过程)。这两个方法分别标示Activity的创建和销毁，并且在Activity整个生命周期中只会调用一次。
  - 可见生存期(Activity在onStart()方法和onStop()方法之间所经历过程)。在该时期内，Activity总是可见的。这两个方法会随着用户的操作可调用多次。
  - 前台生存期(Activity在onResume()，onPause()方法之间所经历过程)。在该时期内，Activity总是处于运行状态，此时的活动是可以和用户进行交互的。这两个方法会随着用户的操作可调用多次。

- 特定操作的方法回调：
    1. 第一次启动(A)Activity，回调：(A)onCreate()->(A)onStart()->(A)onResume();
    2. 用户从(A)Activity打开新的(B)Activity，回调如下：(A)onPause()->(B)onCreate()->(B)onStart()->(B)onResume()->(A)onSaveInstanceState()->(A)onStop();
    3. 从(B)Activity返回(A)Activity,回调如下：(B)onPause()->(A)onRestart()->(A)onStart()->(A)onResume->(B)onStop()->(B)onDestroy();
    4. 用户从(A)Activity打开一个透明主题或是Dialog形式的(B)Activity,回调如下：(A)onPause()->(B)onCreate()->(B)onStart()->(B)onResume();
    5. 从4.的(B)Activity返回（A）Activity，回调如下：(B)onPause()->(A)onResume()->(B)onStop()->(B)onDestroy();
    6. 按home键，(A)Activity返回到桌面时，回调如下：(A)onPause()->(A)onSaveInstanceState()->(A)onStop();
    7. 从桌面返回(A)Activity时，回调如下：(A)onRestart()->(A)onStart()->(A)onResume;
    8. 按back键回退(A)Activity时，回调如下：(A)onPause()->(A)onStop()->(A)onDestroy();
- 异常情况下操作的方法回调
    - 旋转屏幕，使其重建Activity，回调如下：(A)onPause()->(A)onSaveInstanceState()->(A)onStop()->(A)onDestroy()->(重建的A)onCreate()->(重建的A)onStart()->(重建的A)onRestoreInstanceState()->(重建的A)onResume();
- 当我们不想让Activity在异常情况下重建，该怎么办？答案是给Activity指定configChanges属性。在指定属性后，当Activity异常销毁时便不再调用onSaveInstanceState()、onRestoreInstanceState()方法，取而代之会执行onConfigurationChanged()方法。

---

#### Activity的启动模式
应用程序都是由一个或多个Activity组成的，Activity的实例由Android内部的任务栈来管理。栈是一种先进后出的集合。对于Android来说，当前显示的Activity必定位于栈顶，当按回退back键时，栈顶的Activity便会被移除任务栈，位于被移除任务栈的Activity下面的Activity成为新的栈顶，同时显示。  

至于为什么需要划分启动模式，是为了更好地管理Activity，例如：当我们只需要一个Activity只有一个实例，那么启动模式便可以帮助我们实现。

- 四种启动模式
    - standard(标准启动模式)，该模式也是Activity的默认启动模式。在该模式下，每次启动一个Activity都会创建一个新的实例，不管该Activity是否在任务栈中存在。同时，无论谁启动该Activity，该Activity都会进入启动它的Activity所属的任务栈。例如：栈A的Activity启动了一个新的Activity，新的Activity会进入栈A。
    - singleTop(栈顶复用模式)，该模式下启动的Activity，会先判断栈顶该Activity的实例是否存在，若存在，则会重用位于栈顶的实例，并且会调用该实例的onNewIntent()方法将新的Intent对象传递到这个实例中。需要注意的是，在复用实例后，这个Activity的onCreate、onStart不会被重新调用，而是调用的onNewIntent、onResume方法。如果新的Activity的实例不在栈顶，那么任然会创建新的Activity实例。
    - singleTask(栈内复用模式)，该模式下启动Activity，那么栈内只能存在一个该Activity的实例。创建该模式的Activity，首先会在栈内寻找是否有实例，有的话会将该实例顶上的其他Activity移除任务栈（销毁），让该Activity位于栈顶；没有的话则会创建一个实例并放在栈顶。
    - singleInstance(单实例模式)，加强版singleTask，该模式下的Activity只能单独位于一个任务栈中，这个任务栈也只会有这一个Activity。singleTask模式下的Activity可以拥有多个位于不同栈的实例，而singleInstance模式下的Activity就只有一个。

- TaskAffinity和allowTaskReparenting
    - TaskAffinity是标示Activity所需任务栈名字的参数。
        - 在默认情况下，Activity所需任务栈的名字为应用的包名。我们也能为每个Activity设置TaskAffinity属性来自定义任务栈名，但是，TaskAffinity属性值不能与包名重复（要是重复还设置了干啥）。  
        - TaskAffinity一般和singleTask启动模式和allowTaskReparenting属性搭配使用，其他情况下该属性设置了也无用。
        - 在一个包名为“A”的项目中，启动一个TaskAffinity属性为“B（B的结构需要和包名类似，如“xxx.xxx.xxx”）”，启动模式为singleTask的Activity。该Activity首先会判断名称为“B”的任务栈是否存在，存在的话在判断是否有该Activity的实例，实例存在，则将该实例置于栈顶；不存在则创建一个至于栈顶。若任务栈不存在，则创建任务栈，再创建实例。
    
    - allowTaskReparenting属性常用于和TaskAffinity属性搭配使用，主要功能界定是栈A中的Activity是否能迁移到栈B中去，而迁移跟Activity的TaskAffinity有关。举个栗子：应用A有甲Activity，应用B有乙Activity，应用A启动应用B的乙。这时，应用A的栈中有甲乙两个Activity，按home回到桌面，启动应用B，这时会发现，应用A的栈中的乙去到了应用B的栈。当然，这是allowTaskReparenting属性为true的情况，如果为false，则应用A的栈不变，应用B的栈内会重新生成一个新的乙。  
    [大佬博客更详细描写](https://blog.csdn.net/javazejian/article/details/52072131)

- 启动模式的设置
    - 通过AndroidMenifest为Activity指定启动模式：
        
        ``` XML
        <activity android:name=".Activity_A"
            android:launchMode="singleTask"
            android:allowTaskReparenting="flase"
            android:taskAffinity="com.angki.a"/>
            
        android:launchMode: 设置启动模式
        android:taskAffinity：设置taskAffinity
        android:allowTaskReparenting：设置allowTaskReparenting
        ```
    - 通过在Intent设置标志位来指定启动模式：

        ``` JAVA
        Intent intent = new Intent(MainActivity.this, Activity_A.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);    
        ```
    
    - 两种启动方式的区别，第二种优先级高于第一种，当两种方式同时存在时，以第二种方式为准。
    
- 常用标记位
    - FLAG_ACTIVITY_NEW_TASK，新建一个Task来启动Activity，和清单文件指定SingleTask启动模式效果相同。
    - FLAG_ACTIVITY_SINGLE_TOP，和清单文件指定SingleTop启动模式效果相同。
    - FLAG_ACTIVITY_CLEAR_TOP，清除该启动Activity同栈中所有位于它上面的Activity，和清单文件指定SingleTask启动模式效果相同。
    - FLAG_ACTIVITY_NO_HISTORY，以该模式启动的Activity在启动其他Activity会自己销毁自己。
    - FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS，使用该模式启动的Activity不添加到最近应用列表，与属性android:excludeFromRecents="true"效果相同。需要注意的是，要想应用列表不显示该Activity，需要一个不同的栈来存放。因为该应用列表不显示是把该Activity所在任务栈置于后台任务栈。举个例子：
        - AB两个Activity同在一个栈内，其中B为FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS模式启动的。当前页面显示B页面，点击多任务管理按键，发现显示的仍是B页面。
        - Main、A两个Activity不在一个栈内，其中A为FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS模式启动的。查看栈显示：
            ```
            Running activities (most recent first):
                TaskRecord{2becf13d #7042 A=com.angki.task U=0 sz=1}
                    Run #1: ActivityRecord{18df41f7 u0 com.angki.androidlearn/.Activity_A t7042}
                TaskRecord{3c3ba83 #7041 A=com.angki.androidlearn U=0 sz=1}
                    Run #0: ActivityRecord{2d080907 u0 com.angki.androidlearn/.MainActivity t7041}

            ```
            点击点击多任务管理按键：
        
            ```
            Running activities (most recent first):
                TaskRecord{3c3ba83 #7041 A=com.angki.androidlearn U=0 sz=1}
                    Run #1: ActivityRecord{2d080907 u0 com.angki.androidlearn/.MainActivity t7041}
                TaskRecord{2becf13d #7042 A=com.angki.task U=0 sz=1}
                    Run #0: ActivityRecord{18df41f7 u0 com.angki.androidlearn/.Activity_A t7042}

            ```
            可以发现，A的栈置于后，当前显示的是Main页面。
        - ABC三个Activity，A属于栈a，BC属于栈b，其中B为FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS模式启动的。当前显示C页面，点击点击多任务管理按键，当前显示A页面，说明栈b被后置。按下back键，页面显示C页面，再按下back键，页面显示B页面，再按下back键，退出程序。
        
#### 参考来源
- [Android开发艺术探索](https://book.douban.com/subject/26599538/)
- [Android开发进阶 从小工到专家](http://item.jd.com/11880368.html)
