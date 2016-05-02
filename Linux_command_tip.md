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
法二：利用 find -newer 與 !-newer 找出借於 new_time_file 時間 (mtime) 與 old_time_file 時間之間的檔案，再進行複製。
```bash
[root@Server]# find /YOUR/DIR/PATH -newer new_time_file ! -newer old_time_file|xargs -i cp {} /YOUR/DEST/DIR/PATH
```
## Tip3	將 timestamp 轉換成日期時間
##### 需求
將 timestamp 轉換成日期時間
##### 指令
利用 date -d 參數轉換 timestamp，記得 timestamp 前要加 @ 符號。
```bash
[root@Server]# date -d @timestamp
```

## Tip4	利用多核處理器將CSV檔壓縮成gzip檔案
##### 參考資料
[GNU Parallel](http://www.gnu.org/software/parallel/)
##### 需求
因 CSV 檔眾多，單純下 gzip 指令壓縮較為耗時，利用 parallel 工具利用多核處理器資源進行壓縮處理。
##### 指令
先遞迴取得 CSV 檔案名稱，再利用 parallel 參數 --pipe 代表將檔案分段拆開給不同 CPU 處理；參數 --recend 表將大檔案切割做後續處理，後面可接切割的標註文字，在這不指定切割點，系統預設以 '\n' 做為切割點；參數 -k 表檔案重組順序要依照 input 順序，簡言之 output 順序要與 input 順序相同；參數 gzip 是以 gzip 方式壓縮檔案。
```bash
[root@Server]# cat gzip_parallel.sh
...
for i in $(ls /where/csv_dir/*.csv)
do
cat $i|parallel --pipe --recend '' -k gzip > $i.gz
done
```

## Tip5	find 排除特定資料夾
##### 需求
/home/file{1,2,3}.txt
/home/work/file{1,2,3}.txt
目標只要搜索 /home 下第一階目錄的 file.txt 檔案，排除 /home/work 目錄下的 file.txt
##### 指令
利用 find 指令搜索 /home，參數 -path 指定 /home/work 目錄並用參數 -prune 判斷為資料夾並不進入目錄搜索；參數 -name 搜索 file.txt 類似的檔名並利用參數 -print 印出檔案全名。
```bash
[root@Server]# find /home -path '/home/work' -prune -o -name file*.txt -print 
```

## Tip6	利用nohup背景執行指令
##### 需求
透過SSH連線Linux系統執行指令時，當關閉終端機或網路中斷時，系統會收到HUP（hangup）信號而中斷指令運行。
##### 指令
利用nohup執行指令，讓指令執行過程中忽略HUP信號。(預設系統output會指定至$HOME/nohup.out)
```bash
[root@Server]# nohup YOURCOMMAND
```
