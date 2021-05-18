# Activity的启动流程

## 1. Activity
1.  Activity
    1. startActivity(5616 -> 5617 -> 5643 -> 5660/5664)
    2. startActivityForResult(5315 -> 5320)
2. Instrumentation
    1. execStartActivity(1690 -> 1723)
3. ActivityTaskManager
    1. startActivity(1043 -> 1047)
    2. startActivityAsUser(1047 -> 1068 -> 1072 -> 1077 -> 1100)
4. ActivityStarter
    1. execute(682 -> 647)
5. ActivityStarter.Request
    1. resolveActivity(444 -> 522)
6. ActivityStackSupervisor
    1. resolveActivity(632 -> 658)
7. Handler
    1. sendMessage(634): 这里发送信息是通过Binder形式

## Activity -> ATM/AMS
1. ActivityManagerService
    1. startProcess(20357 -> 20360 -> 20364 -> 20376)
    2. startProcessLocked(3309 -> 3315)
2. ProcessList
    1. startProcessLocked(1816 -> 2042 -> 2061 -> 2105)
    2. startProcess(2309 -> 2417)
3. Process
    1. start(627 -> 649)
4. ZygoteProcess
    1. start(345 -> 372)
    2. startViaZygote(626 -> 795)
    3. zygoteSendArgsAndGetResult(422 -> 460)
    4. attemptZygoteSendArgsAndGetResult(463): 这里发送消息到Zygote是通过Socket形式

## ATM/AMS -> Activity
1. ZygoteInit
    1. main(832 -> 934)
2. ZygoteServer
    1. runSelectLoop(424 -> 546)
3. ZygoteConnect
    1. processOneCommand(121 -> 257)
4. Zygote
    1. forkAndSpecialize(337 -> 344)
    2. nativeForkAndSpecialize(366): native方法, 调用一次返回两次
5. com_android_internal_os_Zygote
    1. com_android_internal_os_Zygote_nativeForkAndSpecialize(2080 -> 2117)
    2. ForkCommon(1098 -> 1138): 真正的fork方法调用, 调用一次, 返回两次
6. ZygoteConnect
    1. processOneCommand(273)
    2. handleChildProc(480->504)
7. ZygoteInit
    1. zygoteInit(991 -> 1002)
8. RuntimeInit
    1. applicationInit(404 -> 422)
    2. findStaticMain(345 -> 380)
9. MethodAndArgsCaller
    1. MethodAndArgsCaller(585)
10. ZygoteInit
    1. main(832 -> 947)
11. MethodAndArgsCaller
    1. run(590->592): 这里执行的是目标App的AtivityThread的main函数

## Activity
1. ActivityThread
    1. main(7608 -> 7642 -> 7643)
    2. attach(7327 -> 7336)
2. ActivityManagerService
    1. attachApplication(5757 -> 5765)
    2. attachApplicationLocked(5315 -> 5685)
3. ActivityTaskManagerService
    1. attachApplication(6884 -> 6890)
4. ActivityStackSupervisor
    1. realStartActivityLocked(718 -> 842 -> 863)
5. TransactionExecutor
    1. execute((69 -> 95 -> 97))
    2. executeCallbacks(104 -> 135)
6. ActivityThread
    1. handleLaunchActivity(3573): 此时App进程已经创建完成, 待启动的Activity也已经做好启动的准备工作了
    
