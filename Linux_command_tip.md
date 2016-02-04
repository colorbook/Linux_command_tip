# Linux Command Tip
---
## 目的
收錄 Linux 操作過程中瑣碎的指令與心得，並附上參考文獻。

## Tip1	固定時間批次刪除檔案
##### 需求
檔案伺服器上需固定時間刪除檔案，騰出硬碟空間。
##### 指令
先找出符合條件的檔案，再利用 xrags 傳遞檔案名稱做為 rm 的參數。注意 atime、ctime 與 mtime 的差別，以及 +、－ 或不加的意義。
```bash
[root@Server]# find /YOUR/DIR/PATH -mtime +30|xargs -n 100 rm -f
```
## Tip2	複製特定時間區間檔案
##### 需求
複製近一個月檔案到特定資料夾中
##### 指令
法一：先找出符合條件的檔案，再利用 xrags 傳遞檔案名稱做為 cp 的參數。注意 xargs 參數 -i 代表將 find 指令的 output 指定到 cp 的指定 input 位置。
```bash
[root@Server]# find /YOUR/DIR/PATH -mtime +30|xargs -i cp {} /YOUR/DEST/DIR/PATH
```
法二：利用find -newer與!-newer找出借於new_time_file時間(mtime)與old_time_file時間之間的檔案，再進行複製。
```bash
[root@Server]# find /YOUR/DIR/PATH -newer new_time_file ! -newer old_time_file|xargs -i cp {} /YOUR/DEST/DIR/PATH
```
