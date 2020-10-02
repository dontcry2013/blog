---
title: MP4 Fast Streaming
date: 2020-04-15 09:41:06
categories: 
tags: [shell]
---
moov position is the key for web friendly mp4, it should be ahead of mdat, the following shell checks the moov position and moves it to the right place.

<!--more-->

``` shell
#!/bin/bash
echo $1
if [[ -z "$1" ]]; then
  echo 'directory is NULL'
  exit
fi

videos=`find $1 -type f -atime -365 | xargs -P 10 file -iN {} | grep "video/mp4" | awk -F ":" '{print $1}'`
st=$(date '+%s')
for video in $videos
do
{
  echo $video >> $st
  ffmpeg -v trace -i $video 2>&1 | grep -e type:\'mdat\' -e type:\'moov\' >> $st
  # if ffmpeg version is under 4
  # qtfaststart -l $line | grep 'mdat\|moov' >> $st
} &
wait
done

# if contain multi mdat process
v_pretask='pretask'
v_task='task'
cat $st | grep -A 1 $1 >> $v_pretask
cat pretask | grep -B 1 -e type:\'mdat\' | grep $1 >> $v_task

# move moov
prefix='NEW-'
while IFS= read -r line
do
  qt-faststart $line $prefix$line
  mv -f $prefix$line $line
  echo qtfaststart -l $line | grep 'mdat\|moov'
done < "$v_task"

exit

linenumbers=`grep -n moov $st | awk -F ':' '{print$1}'`
touch 'task.txt'
for lnumber in $linenumbers
do
  b=$(( $lnumber % 3 ))
  if [ $b = 0 ]; then
    lin=$(($lnumber - 2))
    echo `head -n $lin $st|tail -n 1` >> task.txt
  fi
done
```