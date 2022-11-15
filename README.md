# redroid



文件目录

root@localhst:~/redroid_frida# ls -al
total 47308
drwxr-xr-x  2 root root     4096 Nov 15 15:49 .
drwx------ 13 root root     4096 Nov 15 15:30 ..
-rwx------  1 root root 48569016 Nov 15 14:38 frida-server-16.0.2-android-arm64
-rw-r--r--  1 root root      464 Nov 15 15:30 frida.rc
-rwx------  1 root root      375 Nov 15 15:02 setup.sh


frida.rc
```
on early-init
        export PATH /sbin:/product/bin:/apex/com.android.runtime/bin:/apex/com.android.art/bin:/system_ext/bin:/system/bin:/system/xbin:/odm/bin:/vendor/bin:/vendor/xbin
        chmod 0700 /frida-server
	chown root root /frida-server
        chmod 0700 /setup.sh
	chown root root /setup.sh
	exec root root -- /setup.sh

service frida /sbin/frida-server -l 0.0.0.0:6666
        user root
	oneshot

on property:sys.boot_completed=1
        start frida
```

setup.sh
```shell
#!/system/bin/sh

tmpPushed=/frida
rm -rf $tmpPushed
mkdir $tmpPushed
cp /frida-server $tmpPushed/frida-server
umount /frida-server ; rm -v /frida-server

# add /sbin
# /sbin/
# ├── frida-server
mkdir /sbin
chown root:root /sbin
chmod 0751 /sbin
cp $tmpPushed/frida-server /sbin/
find /sbin -type f -exec chmod 0755 {} \;
find /sbin -type f -exec chown root:root {} \;
```


docker command
```
docker run -itd --rm --privileged \
  --name frida_server_test \
  -v $PWD/data_frida_server:/data \
  -v $PWD/frida.rc:/vendor/etc/init/frida.rc \
  -v $PWD/frida-server-16.0.2-android-arm64:/frida-server \
  -v $PWD/setup.sh:/setup.sh \
  -p 5555:5555 \
  -p 5556:6666 \
  redroid/redroid:11.0.0-latest \
  androidboot.redroid_gpu_mode=guest \
  ro.secure=0
```
