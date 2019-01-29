#!/bin/bash
# Description : Creating/Removing LVM snapshots (root /var /opt) # Author : Andrzej Hochbaum
# Date : 08.11.2018
# {function 1} Check if OS using LVM
func1 () {
ch_lvm=`awk '/\/dev\/mapper/' /etc/fstab | wc -l` if [ "$ch_lvm" -eq 0 ]; then
         echo "LVM is not in use - exiting.."; exit0;
      fi
}
# List of all lvm devices: /etc/fstab
MAPPER=`awk '/\/dev\/mapper/' /etc/fstab`
# {function 2} Creating LVM Snapshots func2 () {
for fs in / /var /opt ; do
if [[ "$MAPPER" =~ " $fs " ]] ; then
         echo "->  Creating snapshot FS $fs.."
# Definition of variables
FULL_PATH=`echo "$MAPPER" | awk -v par=$fs '/\'$fs\ '/ {print $1}'`
VGPATH=`echo $FULL_PATH | awk -F"-" '{print $1}'`
RLVS=`echo $FULL_PATH | awk -F"-" '{out = ""; for(i=2; i <= NF; i++) out=out $i " "; print out}'` VGFREE=`vgs --units m --rows "$VGPATH" | awk -F"." '/VFree/ {gsub("VFree",""); gsub(" ",""); {print $1}}'`
# Check available space on VG group and create snapshot if [ $VGFREE -lt 1000 ] ; then
echo "-> There are not available space to create LVM $fs Snapshot.."
echo "-> Current available space: $VGFREE Megabytes" else
               NAME=`echo ${fs/\//}`
lvcreate --size "$size" --snapshot --name "$NAME"_snapshoot_INF "$FULL_PATH" fi
fi
echo "" done
}
# {function 3} Removing LVM Snapshots
func3 () {
LVM_SNAP=`lvdisplay -c | awk -F":" '/snapshoot_INF/ {gsub(" ",""); {print $1}}'`
if [[ -z "$LVM_SNAP" ]] ; then
echo "System does not have snapshoots"
else
for sn in `echo "$LVM_SNAP" | awk -F":" '/snapshoot_INF/ \ {gsub(" ",""); {print $1}}'` ; do lvremove -y $sn ; done
fi }
case "$1" in 'create')
        if [[ "$2" == "" ]] ; then
           size="1G"
           func1
           func2
	   exit 0;
         else
           if [[ "$2" =~ ^[200-900]+$ ]] && [ "$2" -gt 200 -a "$2" -lt 990 ]; then
              size="$2m"
              func1
              func2
              exit 0;
else echo "Size is not belongs to available range of integer" >&2 && exit 1; fi fi
;;
'remove')
        func3
;; *)
        echo "Script for create/remove system's LVM Snapshots"
        echo "Usage: $0 { create | remove }
[200-900] /"
exit 1
;; esac
/Default size of LVS is 1G - if you want to change type: $0 create
SCRIPT :
#!/bin/bash
# Description : Creating/Removing LVM snapshots (root /var /opt) # Author : Andrzej Hochbaum
# Date : 08.11.2018
# {function 1} Check if OS using LVM func1 () {
ch_lvm=`awk '/\/dev\/mapper/' /etc/fstab | wc -l` if [ "$ch_lvm" -eq 0 ]; then
echo "LVM is not in use - exiting.."; exit0; fi
}
# List of all lvm devices: /etc/fstab MAPPER=`awk '/\/dev\/mapper/' /etc/fstab`
# {function 2} Creating LVM Snapshots func2 () {
for fs in / /var /opt ; do
if [[ "$MAPPER" =~ " $fs " ]] ; then
echo "-> Creating snapshot FS $fs.."
# Definition of variables
FULL_PATH=`echo "$MAPPER" | awk -v par=$fs '/\'$fs\ '/ {print $1}'`
VGPATH=`echo $FULL_PATH | awk -F"-" '{print $1}'`
RLVS=`echo $FULL_PATH | awk -F"-" '{out = ""; for(i=2; i <= NF; i++) out=out $i " "; print out}'` VGFREE=`vgs --units m --rows "$VGPATH" | awk -F"." '/VFree/ {gsub("VFree",""); gsub(" ",""); {print $1}}'`
# Check available space on VG group and create snapshot if [ $VGFREE -lt 1000 ] ; then
echo "-> There are not available space to create LVM $fs Snapshot.."
echo "-> Current available space: $VGFREE Megabytes" else
NAME=`echo ${fs/\//}`
lvcreate --size "$size" --snapshot --name "$NAME"_snapshoot_INF "$FULL_PATH" fi
fi
echo "" done
}
# {function 3} Removing LVM Snapshots func3 () {
LVM_SNAP=`lvdisplay -c | awk -F":" '/snapshoot_INF/ {gsub(" ",""); {print $1}}'`
if [[ -z "$LVM_SNAP" ]] ; then
echo "System does not have snapshoots"
else
for sn in `echo "$LVM_SNAP" | awk -F":" '/snapshoot_INF/ \ {gsub(" ",""); {print $1}}'` ; do lvremove -y $sn ; done
fi
}
case "$1" in 'create')
if [[ "$2" == "" ]] ; then size="1G"
func1
func2
exit 0; else
if [[ "$2" =~ ^[200-900]+$ ]] && [ "$2" -gt 200 -a "$2" -lt 990 ]; then size="$2m"
func1
func2
exit 0;
else echo "Size is not belongs to available range of integer" >&2 && exit 1; fi
fi ;;
'remove') func3
;;
*)
echo "Script for create/remove system's LVM Snapshots"
echo "Usage: $0 { create | remove } exit 1
;;
esac
