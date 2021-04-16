# 关于编译ROM相关操作
1. 首先按照[新人上手](https://wiki.n.miui.com/pages/viewpage.action?pageId=145497691)中的步骤下载好源码
2. 然后执行source build/envsetup.sh, 作用: 初始化环境变量(猜的)
3. 执行lunch 机型命令(以我手中的J7B为例, lunch bomb-userdebug, 后面这个可以是user/userdebug/eng, 机型信息可以在[这个网站获得](https://husky.pt.miui.com/device/info))
4. 准备好前两步就可以开始编译了
    * 编译framework: make framework-minus-apex
    * 编译services: make services
    * 编译system.img: make systemimage -j12
5. 注意, 版本不一致时, 刷入framework和services会导致手机卡第二屏, 在刷入system.img是要在fastbootd模式下
    * 进入fastboot模式: 音量- + 开机键
    * 进入fastbootd模式: adb reboot fastboot

## 参考
* [新人上手指南](https://wiki.n.miui.com/pages/viewpage.action?pageId=145497691)
* [Android R Framework 变化](https://wiki.n.miui.com/pages/viewpage.action?pageId=559141717)