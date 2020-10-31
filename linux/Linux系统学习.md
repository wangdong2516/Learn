<center><h1> Linux系统学习

> 设备挂载

1. `mount`： 查看系统中已经进行挂载的设备。

2. `mount -a`:根据/etc/fstab配置文件中的内容，进行自动挂载，所有的内容都需要挂载一遍。

3. `mount -t[指定文件系统] -o[特殊选项] 设备文件名 挂载点`：

    + 选项介绍：-t表示文件系统，加入文件系统类型来指定挂载的类型，可以使用ext3、ext4，iso9660等文件系统。

    + -o选项：可以指定挂载的额外选项。

        atime/noatime:更新访问时间/不更新访问时间，访问分区的文件的时候，是否更新文件的访问时间，默认是更新。

        async/sync：异步/同步，默认是异步

        auto/noauto：自动/手动。mount -a命令执行的时候，是否会自动安装/etc/fstad文件内容挂载，默认是自动

        defaults:定义默认值，相当于rw,suid,dev,exec,auto,nouser,async这七个选项

        exec/noexec：执行/不执行，设定是否允许在文件系统中执行可执行文件，默认是执行

        remount：更新挂载已经挂载的文件系统，一般用于指定修改特殊权限。

        rw/ro：读写/只读，文件系统挂载的时候，是否具有读写的权限

        suid/nosuid：具有/不具有SUID权限，设置文件系统是否具备SUID权限和SGID权限，默认是具有

        user/nouser:允许/不允许普通用户进行挂载操作，设置文件系统是否允许普通用户挂载，默认是不允许，只有root用户可以进行挂载。

        usrquota：写入代表文件系统支持用户催盘配额，默认不支持

        grpquota:写入代表文件系统支持组磁盘配额，默认不支持。

4. `挂载光盘`:

    + 建立挂载点：mkdir /mnt/cdrom/
    + 挂载光盘: mount -t iso9960 /dev/cdrom /mnt/cdrom/

5. `卸载光盘`: umount 设备文件名/挂载点，卸载命令不能`省略`