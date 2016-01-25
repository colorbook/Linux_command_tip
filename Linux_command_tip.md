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

