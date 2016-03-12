
HAL: lights.c

把新文件上传到服务器, 所在目录:
hardware/libhardware/modules/lights/led_hal.c
hardware/libhardware/modules/lights/Android.mk

Android.mk内容如下:
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := lights.tiny4412
LOCAL_MODULE_RELATIVE_PATH := hw
LOCAL_C_INCLUDES := hardware/libhardware
LOCAL_SRC_FILES := lights.c
LOCAL_SHARED_LIBRARIES := liblog
LOCAL_MODULE_TAGS := eng
include $(BUILD_SHARED_LIBRARY)


修改
vi vendor/friendly-arm/tiny4412/device-tiny4412.mk 这个文件

搜索lights (/lights),将代码注释掉
ifeq ($(BOARD_USES_PWMLIGHTS),false)
#PRODUCT_COPY_FILES += \
#       $(VENDOR_PATH)/proprietary/lights.tiny4412.so:system/lib/hw/lights.tiny4412.so
endif


编译：
root@book-virtual-machine:/work/android-5.0.2#
# . setenv
# lunch
输入“full_tiny4412-eng”之前的数字标号15
$ mmm hardware/libhardware/modules/lights -B
$ make snod
$ ./gen-img.sh


内核也需要修改：
1. drivers/leds/led-class.c: 0644 改为 0666
static struct device_attribute led_class_attrs[] = {
	__ATTR(brightness, 0666, led_brightness_show, led_brightness_store),
	__ATTR(max_brightness, 0444, led_max_brightness_show, NULL),
#ifdef CONFIG_LEDS_TRIGGERS
	__ATTR(trigger, 0666, led_trigger_show, led_trigger_store),
#endif
	__ATTR_NULL,
};

2. drivers/leds/ledtrig-timer.c: 0644 改为 0666
#if defined(CONFIG_MACH_IPCAM)
static DEVICE_ATTR(delay_on, 0666, led_delay_on_show, led_delay_on_store);
static DEVICE_ATTR(delay_off, 0666, led_delay_off_show, led_delay_off_store);
#else
static DEVICE_ATTR(delay_on, 0666, led_delay_on_show, led_delay_on_store);
static DEVICE_ATTR(delay_off, 0666, led_delay_off_show, led_delay_off_store);
#endif

make zImage


验证厂家提供的lights.tiny4412.so与刚编译出来的lights.tiny4412.so是否一致。
用这个命令确保我们提供的lights.c已经编进system.img:
diff vendor/friendly-arm/tiny4412/proprietary/lights.tiny4412.so out/target/product/tiny4412/system/lib/hw/lights.tiny4412.so


logcat lights:V *:S

上传本次版本代码到GitHub：
0.git init (第一次执行即可)
1.git add -A
2.git commit -m "v1, Hal of Lights"
3.git tag v1
4.git remote add origin https://github.com/hujinfan/SYS_0002_Lights.git （第一次执行即可）
5.git push -u origin master
6.git push origin --tags


