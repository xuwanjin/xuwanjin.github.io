---
layout:     post
title:      "Android Wear OTA 升级"
subtitle:   "记一次 Android OTA 代码合入"
date:       2019-01-25 20:30:40
author:     "Mathew"
catalog: true
header-img: "img/post-bg-2017.jpg"
tags:
    - Android
    - OTA
---

# Android Wear OTA 升级

[TOC]

设备添加后有7天的有效期， 升级前先确认是否设备在有效期之内。
同时需要给每个设备添加设别的MID, 每个设备的MID都不一样

### 错误 一
```
[    2.185290] Supported API: 3
[    2.195030] charge_status 3, charged 0, status 0, capacity 95
[    2.797626] Finding update package...
[    2.857440] I:Update location: @/cache/recovery/block.map
[    2.857510] Opening update package...
[    2.892843] sysutil: mmapped 1 ranges
[    2.893456] I:read key e=3 hash=20
[    2.893483] I:1 key(s) loaded from /res/keys
[    2.893519] Verifying update package...
[    2.929443] I:comment is 1738 bytes; signature 1720 bytes from end
[    2.957725] I:signature (offset: 0xe01bf, length: 1714): 308206ae06092a864886f70d010702a082069f3082069b020101310b300906052b0e03021a0500300b06092a864886f70d010701a08204ac308204a830820390a003020102020900936eacbe07f201df300d06092a864886f70d0101050500308194310b3009060355040613025553311330110603550408130a43616c69666f726e6961311630140603550407130d4d6f756e7461696e20566965773110300e060355040a1307416e64726f69643110300e060355040b1307416e64726f69643110300e06035504031307416e64726f69643122302006092a864886f70d0109011613616e64726f696440616e64726f69642e636f6d301e170d3038303232393031333334365a170d3335303731373031333334365a308194310b3009060355040613025553311330110603550408130a43616c69666f726e6961311630140603550407130d4d6f756e7461696e20566965773110300e060355040a1307416e64726f69643110300e060355040b1307416e64726f69643110300e06035504031307416e64726f69643122302006092a864886f70d0109011613616e64726f696440616e64726f69642e636f6d30820120300d06092a864886f70d01010105000382010d00308201080282010100d6931904dec60b24b1edc762e0d9d8253e3ecd6ceb1de2ff068ca8e8bca8cd6bd3786ea70aa76ce60ebb0f993559ffd93e77a943e7e83d4b64b8e4fea2d3e656f1e267a81bbfb230b578c20443be4c7218b846f5211586f038a14e89c2be387f8ebecf8fcac3da1ee330c9ea93d0a7c3dc4af350220d50080732e0809717ee6a053359e6a694ec2cb3f284a0a466c87a94d83b31093a67372e2f6412c06e6d42f15818dffe0381cc0cd444da6cddc3b82458194801b32564134fbfde98c9287748dbf5676a540d8154c8bbca07b9e247553311c46b9af76fdeeccc8e69e7c8a2d08e782620943f99727d3c04fe72991d99df9bae38a0b2177fa31d5b6afee91f020103a381fc3081f9301d0603551d0e04160414485900563d272c46ae118605a47419ac09ca8c113081c90603551d230481c13081be8014485900563d272c46ae118605a47419ac09ca8c11a1819aa48197308194310b3009060355040613025553311330110603550408130a43616c69666f726e6961311630140603550407130d4d6f756e7461696e20566965773110300e060355040a1307416e64726f69643110300e060355040b1307416e64726f69643110300e06035504031307416e64726f69643122302006092a864886f70d0109011613616e64726f696440616e64726f69642e636f6d820900936eacbe07f201df300c0603551d13040530030101ff300d06092a864886f70d010105050003820101007aaf968ceb50c441055118d0daabaf015b8a765a27a715a2c2b44f221415ffdace03095abfa42df70708726c2069e5c36eddae0400be29452c084bc27eb6a17eac9dbe182c204eb15311f455d824b656dbe4dc2240912d7586fe88951d01a8feb5ae5a4260535df83431052422468c36e22c2a5ef994d61dd7306ae4c9f6951ba3c12f1d1914ddc61f1a62da2df827f603fea5603b2c540dbd7c019c36bab29a4271c117df523cdbc5f3817a49e0efa60cbd7f74177e7a4f193d43f4220772666e4c4d83e1bd5a86087cf34f2dec21e245ca6c2bb016e683638050d2c430eea7c26a1c49d3760a58ab7f1a82cc938b4831384324bd0401fa12163a50570e684d318201ca308201c60201013081a2308194310b3009060355040613025553311330110603550408130a43616c69666f726e6961311630140603550407130d4d6f756e7461696e20566965773110300e060355040a1307416e64726f69643110300e060355040b1307416e64726f69643110300e06035504031307416e64726f69643122302006092a864886f70d0109011613616e64726f696440616e64726f69642e636f6d020900936eacbe07f201df300906052b0e03021a0500300d06092a864886f70d0101010500048201002f530a8c9aad9193a54f904cd17725268d634c52f83d8a220fbcb06243d42c34ff248e31bbbbe4936960cf537e588fa23886e6a0826606f76b2f61d72e80d5dca92e67852d133c09e6ceb71e6e642abacf852fde9d055b7071657365b420d58301eaa48b92bd55ef8d56ea75af7b5ff6a90180cfecb770aa3c346a6adeba2856faef3467497cdd7ee9b2b13dda59ff1c9dfb61f3783846c7add69f657c37b1f6120bf3ff5748ccde1ac7bcce9501c9f3f038a1020cd219dfb0c7b0c4716faa075abf47f6b267e0221e1ebcaff948eb69f8d680890fc8ec19bdaeb424f162be6a2b8e1cd1dafe0e69a6fddaa6801440d811bd09375e99eaac16007dc664981669
[    2.958260] I:whole-file signature verified against RSA key 0
[    2.958290] Update package verification took 0.0 s (result 0).
[    2.974081] Installing update...
[    2.991207] E:Failed to parse build number in post-build-incremental=NMF26F.
[    3.023978] E:Failed to parse build number in pre-build-incremental=NMF26F.
[    3.077073] sysutil: mmapped 1 ranges
[    3.082298] Source: zwd503/zwd503/zwd503:7.1.1/NMF26F/NMF26F:user/test-keys
[    3.082871] Target: zwd503/zwd503/zwd503:7.1.1/NMF26F/NMF26F:user/test-keys
[    3.083040] Verifying current system...
[    3.571328] partition read matched size 12168488 sha 9be315439d64975a05e63ef1b90920cc713b677a
[    3.577771] 60440576 bytes free on /cache (12168488 needed)
[    3.577810] Verifying radio-update...
[    3.601081] contents of partition "/dev/block/bootdevice/by-name/aboot" didn't match EMMC:/dev/block/bootdevice/by-name/aboot:502988:bca230740ec341960d874c8248658bbeb03bf77c:502988:875221a017c6de044015c4c2583b0f990e709d93
[    3.601162] file "EMMC:/dev/block/bootdevice/by-name/aboot:502988:bca230740ec341960d874c8248658bbeb03bf77c:502988:875221a017c6de044015c4c2583b0f990e709d93" doesn't have any of expected sha1 sums; checking cache
[    3.601239] failed to stat "/cache/saved.file": No such file or directory
[    3.601261] failed to load cache file
[    3.601280] script aborted: E3005: "EMMC:/dev/block/bootdevice/by-name/aboot:502988:bca230740ec341960d874c8248658bbeb03bf77c:502988:875221a017c6de044015c4c2583b0f990e709d93" has unexpected contents.
[    3.673619] E:Error in @/cache/recovery/block.map
[    3.673830] (Status 7)
[    3.690180]
[    3.725739] I:@/cache/recovery/block.map
[    3.725769] 0
[    3.725783] time_total: 1
[    3.725795] retry: 0
[    3.725807] error: 3005
[    3.725818] uncrypt_time: 0
[    3.725892] Installation aborted.
[    3.940009] I:Saving locale "zh_CN"
[   64.439861] I:Saving locale "zh_CN"
```
典型的错误， 一般表示你的机器里文件不对

### 错误二
```
[    3.313200] E:Failed to parse build number in post-build-incremental=NMF26F.
[    3.346909] E:Failed to parse build number in pre-build-incremental=NMF26F.
[    3.409205] sysutil: mmapped 1 ranges
[    3.448946] Source: zwd503/zwd503/zwd503:7.1.1/NMF26F/NMF26F:userdebug/test-keys
[    3.449077] Target: zwd503/zwd503/zwd503:7.1.1/NMF26F/NMF26F:userdebug/test-keys
[    3.449174] Verifying current system...
[    3.451479] failed to stat "/system/bin/install-recovery.sh": No such file or directory
[    3.451570] file "/system/bin/install-recovery.sh" doesn't have any of expected sha1 sums; checking cache
[    3.451732] failed to stat "/cache/saved.file": No such file or directory
[    3.451835] failed to load cache file
[    3.451919] script aborted: E3005: "/system/bin/install-recovery.sh" has unexpected contents.
[    3.782245] E:Error in @/cache/recovery/block.map
[    3.782288] (Status 7)
[    3.809853]
[    3.833861] I:@/cache/recovery/block.map
[    3.833894] 0
[    3.833910] time_total: 0
[    3.833925] retry: 0
[    3.833939] error: 3005

```
可以在recovery里的代码里找到关于status 7 是什么错误的描述

### 错误三

```
[  142.396029] .../dev/block/bootdevice/by-name/system image recovered successfully.
[  143.209470] performing verification
[  143.223947] blockimg version is 4
[  143.224015] maximum stash entries 49
[  143.224039] creating stash /cache/recovery/2bdde8504898ccfcd2c59f20bb8c9c25f73bb524/
[  143.224524] 60276736 bytes free on /cache (8466432 needed)
[  144.607697] failed to verify blocks (expected 3922763c95ae07b0b50ff2367f0acd342de20687, read 7654819d85333242e71053117738a7d58c859758)
[  144.607817] partition has unexpected contents
[  144.614868] failed to read blocks for move
[  144.615011] failed to execute command [move 3922763c95ae07b0b50ff2367f0acd342de20687 2,95617,96531 914 2,280478,281392]
[  144.615435] deleting stash 2bdde8504898ccfcd2c59f20bb8c9c25f73bb524
[  144.681010] script aborted: E1004: system partition fails to recover
[  144.699941] E:Error in @/cache/recovery/block.map
[  144.699989] (Status 7)
[  144.733357]
[  144.755011] I:@/cache/recovery/block.map
[  144.755047] 0
[  144.755063] time_total: 142
[  144.755078] retry: 0
[  144.755092] error: 1004
[  144.755106] uncrypt_time: 0
[  144.755163] Installation aborted.

```

 ``` system partition fails to recover ``` 个说明了系统分区不对， 可能你的刷机的包不对，
或者差分的时候base版本选择的不对， 或者root的情况下adb remount了

### 注意事项

```
1. 不要adb remount
2. 一定要格式化分区，
3. fastboot刷机, 相关的有没有刷， 如： bootloader的emmc_appsboot.mbn
```

```
高通平台的
zwd503-download_files
zwd503-unsparse-download_files//这两个有什么区别
```

这个是第一次make的情况，emmc_appsboot.mbn 只有unsigned的sha1sum值不一样。 

```
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/first_make/zwd503$ find ./ -name emmc_appsboot.mbn |xargs sha1sum 
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-download_files/emmc_appsboot.mbn
db808b0dba96abc5b56c46d84af3be37316085c8  ./unsigned/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-unsparse-download_files/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./signed/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./emmc_appsboot.mbn
```

这是第一次make otapackage 的生成的情况， target_file里的和download ， unsparse download的不一样。 很明显download 和unsparse download是上次编译的情况的emmc_appsboot.mbn
```bash
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/first_make_otapackage/zwd503$ find ./ -name "emmc_appsboot.mbn" | xargs sha1sum
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-download_files/emmc_appsboot.mbn
db808b0dba96abc5b56c46d84af3be37316085c8  ./unsigned/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-unsparse-download_files/emmc_appsboot.mbn
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734  ./signed/emmc_appsboot.mbn
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734  ./emmc_appsboot.mbn
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734  ./obj/PACKAGING/target_files_intermediates/zwd503-target_files-NMF26F/RADIO/emmc_appsboot.mbn
```




```
ubuntu14@ubuntu14:/mnt/code/huaqin/watch_code/out/target/product/zwd503$ find ./ -name "emmc_appsboot.mbn" | xargs sha1sum 
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-download_files/emmc_appsboot.mbn
db808b0dba96abc5b56c46d84af3be37316085c8  ./unsigned/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-unsparse-download_files/emmc_appsboot.mbn
b8df50fb40a1871839a9a400a4e4960f1e9dfe8b  ./signed/emmc_appsboot.mbn
b8df50fb40a1871839a9a400a4e4960f1e9dfe8b  ./emmc_appsboot.mbn
b8df50fb40a1871839a9a400a4e4960f1e9dfe8b  ./obj/PACKAGING/target_files_intermediates/zwd503-target_files-NMF26F/RADIO/emmc_appsboot.mbn

```


### 文件比较
这个时候第一次make的时候生成的文件

```bash
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/first_make/zwd503$ find ./ -name emmc_appsboot.mbn |xargs sha1sum 
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-download_files/emmc_appsboot.mbn
db808b0dba96abc5b56c46d84af3be37316085c8  ./unsigned/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-unsparse-download_files/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./signed/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./emmc_appsboot.mbn
```

```
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/first_make/zwd503$ find ./ -name "emmc_appsboot.mbn" | xargs ls -la
-rwxrwxr-x 1 ubuntu14 ubuntu14 526500 Oct 23 09:45 ./emmc_appsboot.mbn
-rw-rw-r-- 1 ubuntu14 ubuntu14 526500 Oct 23 09:45 ./signed/emmc_appsboot.mbn
-rw-rw-r-- 1 ubuntu14 ubuntu14 522404 Oct 23 09:45 ./unsigned/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 12:20 ./zwd503-download_files/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 12:22 ./zwd503-unsparse-download_files/emmc_appsboot.mbn
```
这是第一次make otapackage 的生成的情况， target_file里的和download ， unsparse download的不一样。 很明显download 和unsparse download是上次编译的情况的emmc_appsboot.mbn。 temp文件是解压出来的


```
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/first_make_otapackage/zwd503$ find ./ -name "emmc_appsboot.mbn" | xargs sha1sum
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-download_files/emmc_appsboot.mbn
db808b0dba96abc5b56c46d84af3be37316085c8  ./unsigned/emmc_appsboot.mbn
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734  ./temp/RADIO/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503-unsparse-download_files/emmc_appsboot.mbn
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734  ./signed/emmc_appsboot.mbn
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734  ./emmc_appsboot.mbn
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734  ./obj/PACKAGING/target_files_intermediates/zwd503-target_files-NMF26F/RADIO/emmc_appsboot.mbn

```

相应的时间戳
```
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/first_make_otapackage/zwd503$ find ./ -name "emmc_appsboot.mbn" | xargs ls -la
-rwxrwxr-x 1 ubuntu14 ubuntu14 526500 Oct 23 14:51 ./bootloader_temp/emmc_appsboot.mbn
-rwxrwxr-x 1 ubuntu14 ubuntu14 526500 Oct 23 12:56 ./emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 13:00 ./obj/PACKAGING/target_files_intermediates/zwd503-target_files-NMF26F/RADIO/emmc_appsboot.mbn
-rw-rw-r-- 1 ubuntu14 ubuntu14 526500 Oct 23 12:56 ./signed/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 12:44 ./temp/RADIO/emmc_appsboot.mbn
-rw-rw-r-- 1 ubuntu14 ubuntu14 522404 Oct 23 12:53 ./unsigned/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 12:53 ./zwd503-download_files/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 12:54 ./zwd503-unsparse-download_files/emmc_appsboot.mbn
```


这是第二次make otapackage 的生成的情况， target_file 里的和download , unsparse download 的不一样。 很明显 download file 和 unsparse download file 是上次编译的情况的 emmc_appsboot.mbn
sha1sum 值很明显就可以看出区别
```
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/second_make_otapackage$ find ./ -name "emmc_appsboot.mbn" | xargs sha1sum
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503/zwd503-download_files/emmc_appsboot.mbn
db808b0dba96abc5b56c46d84af3be37316085c8  ./zwd503/unsigned/emmc_appsboot.mbn
b8df50fb40a1871839a9a400a4e4960f1e9dfe8b  ./zwd503/temp/RADIO/emmc_appsboot.mbn
6dbfbf634f1f878d2c7ee3af2332815c1ec40656  ./zwd503/zwd503-unsparse-download_files/emmc_appsboot.mbn
b8df50fb40a1871839a9a400a4e4960f1e9dfe8b  ./zwd503/signed/emmc_appsboot.mbn
b8df50fb40a1871839a9a400a4e4960f1e9dfe8b  ./zwd503/emmc_appsboot.mbn
b8df50fb40a1871839a9a400a4e4960f1e9dfe8b  ./zwd503/obj/PACKAGING/target_files_intermediates/zwd503-target_files-NMF26F/RADIO/emmc_appsboot.mbn
```

这个是第二次 make otapackage 的时间戳
```
ubuntu14@ubuntu14:/mnt/code/huaqin/out_10_23/second_make_otapackage$ find ./ -name "emmc_appsboot.mbn" | xargs ls -la
-rwxrwxr-x 1 ubuntu14 ubuntu14 526500 Oct 23 13:30 ./zwd503/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 13:43 ./zwd503/obj/PACKAGING/target_files_intermediates/zwd503-target_files-NMF26F/RADIO/emmc_appsboot.mbn
-rw-rw-r-- 1 ubuntu14 ubuntu14 526500 Oct 23 13:30 ./zwd503/signed/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 13:14 ./zwd503/temp/RADIO/emmc_appsboot.mbn
-rw-rw-r-- 1 ubuntu14 ubuntu14 522404 Oct 23 13:26 ./zwd503/unsigned/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 13:26 ./zwd503/zwd503-download_files/emmc_appsboot.mbn
-rw-r--r-- 1 ubuntu14 ubuntu14 526500 Oct 23 13:27 ./zwd503/zwd503-unsparse-download_files/emmc_appsboot.mbn
```

```
862ab299ee05949c41cb7f610b91894b98aba3f6

//上次版本的该分区的sha1sum值                  目标版本的该分区的sha1sum值
3b1b877a35d9e4dad77af1ca6e87b6817ce2c734   b8df50fb40a1871839a9a400a4e4960f1e9dfe8b
```


###  解决思路
```bash
ubuntu14@ubuntu14:/mnt/code/huaqin/watch_code_02/code$ mgrep otapackage
./device/qcom/msm8909w/common/generate_extra_images.mk:430:.PHONY: otapackage
./device/qcom/msm8909w/common/generate_extra_images.mk:431:otapackage: $(INSTALLED_SEC_BOOTIMAGE_TARGET) $(INSTALLED_SEC_RECOVERYIMAGE_TARGET)
./build/core/Makefile:1966:.PHONY: otapackage
./build/core/Makefile:1967:otapackage: $(INTERNAL_OTA_PACKAGE_TARGET)
./vendor/huaqin/build/tasks/generate_extra_images.mk:359:ifneq (,$(filter target-files-package otapackage dist,$(MAKECMDGOALS)))
./vendor/qcom/build/tasks/generate_extra_images.mk:430:.PHONY: otapackage
./vendor/qcom/build/tasks/generate_extra_images.mk:431:otapackage: $(INSTALLED_SEC_BOOTIMAGE_TARGET) $(INSTALLED_SEC_RECOVERYIMAGE_TARGET)
./vendor/qcom/proprietary/common/scripts/Android.mk:89:otapackage: gensecimage_target
./vendor/qcom/proprietary/common/scripts/Android.mk:92:otapackage: gensecimage_target gensecimage_install
```

在这里makefile做了处理， 替换掉download_file， unsparse_download_file里的文件
```makefile
.PHONY: replace-images-from-target-file
replace-images-from-target-file: $(BUILT_TARGET_FILES_PACKAGE)
	$(call pretty,"Update images from target-file")
	$(hide) python vendor/huaqin/build/scripts/replace_img_from_target_files.py $(BUILT_TARGET_FILES_PACKAGE) $(PRODUCT_OUT)

# ensure images is same with images inside target-file
ifneq (,$(filter target-files-package otapackage dist,$(MAKECMDGOALS)))
$(BUILT_DOWNLOAD_FILES_PACKAGE): replace-images-from-target-file
endif

.PHONY: download-files-package
download-files-package: $(BUILT_DOWNLOAD_FILES_PACKAGE)
download-files-package: $(BUILT_UNSPARSE_DOWNLOAD_FILES_PACKAGE)

ifneq ($(strip $(USE_PREBUILT_NON_HLOS_BINARIES)),true)
download-files-package: $(BUILT_NON_HLOS_SYMBOLS_PACKAGE)
endif
```


一次编译生成的out目录。 注意时间戳
```bash
ubuntu14@ubuntu14:/mnt/code/huaqin/watch_code_3/out/target/product/zwd503$ ls -lah
total 4.2G
drwxrwxr-x 18 ubuntu14 ubuntu14 4.0K Oct 25 12:44 .
drwxrwxr-x  3 ubuntu14 ubuntu14 4.0K Oct 25 09:53 ..
-rw-rw-r--  1 ubuntu14 ubuntu14   14 Oct 25 09:55 android-info.txt
-rw-rw-r--  1 ubuntu14 ubuntu14  12M Oct 25 12:42 boot.img
-rw-rw-r--  1 ubuntu14 ubuntu14   56 Oct 25 12:36 build_fingerprint.txt
drwxrwxr-x  2 ubuntu14 ubuntu14 4.0K Oct 25 11:45 cache
-rw-r--r--  1 ubuntu14 ubuntu14 5.1M Oct 25 12:42 cache.img
-rw-rw-r--  1 ubuntu14 ubuntu14  70K Oct 25 09:53 clean_steps.mk
-rw-r--r--  1 ubuntu14 ubuntu14 161K Oct 25 09:58 cmnlib.mbn
drwxrwxr-x  6 ubuntu14 ubuntu14 4.0K Oct 25 11:45 data
drwxrwxr-x  3 ubuntu14 ubuntu14 4.0K Oct 25 10:26 dex_bootjars
-rwxrwxr-x  1 ubuntu14 ubuntu14 486K Oct 25 12:37 emmc_appsboot.mbn
drwxrwxr-x  2 ubuntu14 ubuntu14 4.0K Oct 25 11:44 fake_packages
-rw-rw-r--  1 ubuntu14 ubuntu14 3.0K Oct 25 09:55 filesmap
-rwxrwxr-x  1 ubuntu14 ubuntu14  63K Oct 25 09:57 fs_image.tar.gz.mbn.img
drwxrwxr-x  5 ubuntu14 ubuntu14 4.0K Oct 25 11:09 gen
-rw-rw-r--  1 ubuntu14 ubuntu14 105K Oct 25 12:38 installed-files.txt
-rw-rw-r--  1 ubuntu14 ubuntu14  11M Oct 25 12:38 kernel
-rw-r--r--  1 ubuntu14 ubuntu14 148K Oct 25 09:57 keymaster.mbn
-rw-rw-r--  1 ubuntu14 ubuntu14 1.3M Oct 25 11:45 module-info.json
-rw-r--r--  1 ubuntu14 ubuntu14  45M Oct 25 10:42 NON-HLOS.bin
drwxrwxr-x 18 ubuntu14 ubuntu14 4.0K Oct 25 11:45 obj
drwxrwxr-x  3 ubuntu14 ubuntu14 4.0K Oct 25 11:44 persist
-rw-r--r--  1 ubuntu14 ubuntu14 4.5M Oct 25 11:44 persist.img
-rw-rw-r--  1 ubuntu14 ubuntu14  965 Oct 25 09:53 previous_build_config.mk
-rw-r--r--  1 ubuntu14 ubuntu14 273K Oct 25 11:45 prog_emmc_firehose_8909w_ddr.mbn
-rw-rw-r--  1 ubuntu14 ubuntu14 1.3M Oct 25 09:55 ramdisk.img
-rw-rw-r--  1 ubuntu14 ubuntu14 3.6M Oct 25 12:38 ramdisk-recovery.img
drwxrwxr-x  3 ubuntu14 ubuntu14 4.0K Oct 25 09:59 recovery
-rw-rw-r--  1 ubuntu14 ubuntu14   67 Oct 25 12:38 recovery.id
-rw-rw-r--  1 ubuntu14 ubuntu14  14M Oct 25 12:42 recovery.img
drwxrwxr-x 16 ubuntu14 ubuntu14 4.0K Oct 25 09:55 root
-rw-r--r--  1 ubuntu14 ubuntu14 159K Oct 25 09:58 rpm.mbn
-rw-r--r--  1 ubuntu14 ubuntu14 167K Oct 25 09:57 sampleapp.mbn
-rw-r--r--  1 ubuntu14 ubuntu14 240K Oct 25 10:04 sbl1.mbn
-rw-r--r--  1 ubuntu14 ubuntu14   80 Oct 25 11:45 sec.dat
-rw-rw-r--  1 ubuntu14 ubuntu14 5.5K Oct 25 12:37 secimage.log
drwxrwxr-x  2 ubuntu14 ubuntu14 4.0K Oct 25 12:37 signed
drwxrwxr-x  6 ubuntu14 ubuntu14 4.0K Oct 25 11:44 symbols
drwxrwxr-x  2 ubuntu14 ubuntu14 4.0K Oct 25 12:43 symbols-non-hlos
-rw-rw-r--  1 ubuntu14 ubuntu14 284M Oct 25 12:44 symbols-non-hlos.zip
drwxrwxr-x 16 ubuntu14 ubuntu14 4.0K Oct 25 12:36 system
-rw-rw-r--  1 ubuntu14 ubuntu14 977M Oct 25 12:42 system.img
-rw-rw-r--  1 ubuntu14 ubuntu14 1.2G Oct 25 12:44 target_files-package.zip
-rw-r--r--  1 ubuntu14 ubuntu14 517K Oct 25 09:57 tzbsp_no_xpu.mbn
-rw-r--r--  1 ubuntu14 ubuntu14 537K Oct 25 09:58 tz.mbn
drwxrwxr-x  2 ubuntu14 ubuntu14 4.0K Oct 25 11:45 unsigned
-rw-r--r--  1 ubuntu14 ubuntu14  31M Oct 25 12:42 userdata.img
drwxrwxr-x  2 ubuntu14 ubuntu14 4.0K Oct 25 12:42 zwd503-download_files
-rw-rw-r--  1 ubuntu14 ubuntu14 584M Oct 25 12:43 zwd503-download_files.zip
-rw-rw-r--  1 ubuntu14 ubuntu14 573M Oct 25 12:44 zwd503-ota-NMF26F.zip
drwxrwxr-x  2 ubuntu14 ubuntu14 4.0K Oct 25 12:43 zwd503-unsparse-download_files
-rw-rw-r--  1 ubuntu14 ubuntu14 585M Oct 25 12:44 zwd503-unsparse-download_files.zip
```

```
ubuntu14@ubuntu14:/mnt/code/huaqin/watch_code_3/out/target/product/zwd503/obj/PACKAGING/target_files_intermediates$ ls -la
total 1152596
drwxrwxr-x  3 ubuntu14 ubuntu14       4096 Oct 25 12:40 .
drwxrwxr-x 11 ubuntu14 ubuntu14       4096 Oct 25 11:47 ..
drwxrwxr-x  9 ubuntu14 ubuntu14       4096 Oct 25 12:39 zwd503-target_files-NMF26F
-rw-rw-r--  1 ubuntu14 ubuntu14 1180238357 Oct 25 12:42 zwd503-target_files-NMF26F.zip
```
在一次编译otapackage之后会有以下几个文件
对以下这几个文件作出相关的说明
```
1. zwd503-ota-NMF26F.zip // 全包升级的zip文件
2. target_files-package.zip  // 用于差分的文件
3. zwd503-target_files-NMF26F.zip  //在obj/PACKAGING/target_files_intermediates下面
4. target_files-package.zip zwd503-ota-NMF26F.zip   zwd503-unsparse-download_files
zwd503-unsparse-download_files.zip zwd503-download_files  zwd503-download_files.zip
以及out下的文件（包括image, mbn文件）时间戳要保证一致， 不同的文件时间戳可以不一样, 这样才能保证这些文件都是make otapackage 的时候生成的文件， 最关键的是 sha1sum 值， 剩下的都不重要。 时间戳可以方便的看到。
5. 要保证同一个文件的 sha1sum （最后升级用来校验的） 值是一样，unsparse 里被unsparse的文件可以不一样， 但是刷机之后同样可以利用差分包升级
6. target_files-package.zip 其实是部分目录加上ota_target_files.zip 文件， 然而ota_target_files.zip 文件都是大写的目录加上镜像文件， system 目录文件
```

### 操作的流程
```
1. 权限问题， 报了很多SeLinux 权限，添加了
2. 后面分区校验失败的情况。 改用target_files_package.zip里的文件刷机了。一开始的失败也有刷机，刷错了文件（以为自己刷错了文件， 其实不是刷错了， 而是后面的out下的和target文件不一致的）
3. 询问为什么使用target_files_package.zip里面的文件， 说是两种方式都可以。(正常的应该使用out下面的刷机)
4. 发现emmc_appboot.mbn校验失败。（一开始也出现了， 他们把base包里东西删除了， 然后做差分了）
5. 然后发现make otapackage 之后， out下面， download_file, unsparse_download_file下面的文件和target_files-pakage不一致。
6. 想到了把target_files-package.zip里的image文件替换掉download_file， unsparse_download_file下的文件
7. 搜索mgrep otapackage的时候， 发现华勤的vendor/huaqin/build/tasks/generate_extra_images.mk 里意境做了处理了
8. 发现了自己的编译方式不太正确， 应该使用./vendor/huaqin/build/build-zwd503-user.sh 至于后面带不带参数，不知道。 看了脚本有对ota参数的处理
9. 采用新的编译编译方式之后发现编译无论如何不能生成target_files-pakage.zip文件。询问相关的人， 给出的结论是我们本来就没有target_files-pakage.zip， 直接用zwd503-ota-NMF26F.zip差分的。
10. 怀疑需要改掉工具了， 但是对比了一下文件，zwd503-ota-NMF26F.zip 这个文件和target_files-pakage.zip文件不一样。zwd503-ota-NMF26F.zip  只是target_files-pakage.zip下的带ota的zip文件
11. 认为我们的编译脚本有问题。编译并没生成相关的target_files-pakage.zip。 
12. 中间想到参考一下POS机器的代码， 看看别人代码的ota是这么实现的。 发现没有target_files-package.zip
13. 这时候询问了一些人， 在编译脚本的添加了otapackage
14. 同时我把技术支持的人的后来的修改的脚本添加进去了。一致导致编译报错。 删除不存在的文件，出现了错误。 尝试在让makefile忽略掉。 但没有成功。
15. 这时候想起其他人添加编译成功了，和别人不一样地方，是自己应该是添加了广升的脚本， 清楚脚本后面的修改。
16. 生成的时间戳和sha1sum都能一一对应。

```



### 反思
```
1. 修手表了， 机器坏了修， 充电， 低电量不能启动，拔错电源
2. 对环境不熟悉， 也被忽悠了几次。可以刷target包，编译脚本也不知道。按照以前的惯性去做。 感觉周一和周二都是在绕圈子。周三才是回到正轨了。
3. otapackage的编译流程不太熟悉。高通的代码编译生成out的目录，unsparse和 download_file的区别
```


## 参考文献
1. [Android8.0.0-r4的OTA升级--差分包升级](https://blog.csdn.net/nwpushuai/article/details/79703043)