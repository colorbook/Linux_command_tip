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

## Tip7	find 查詢特定時間區間的檔案
##### 參考資料
[find(1)－newerXY](http://linux.die.net/man/1/find)
##### 需求
/home 目錄中每分鐘固定生成檔案，久而久之累積大量檔案資料，在管理上要查詢特定時間之檔案較無效率。
##### 指令
利用 find 指令搜索 /home 目錄下，檔案 mtime 界於 5/5~10 的檔案資料。
```bash
[root@Server]# find /home -type f ! -newermt "2016-05-10" -newermt "2016-05-05"
```

## Tip8	date 查詢150天前的時間
##### 參考資料
[date(1) - Linux man page](http://linux.die.net/man/1/date)
##### 需求
查詢150天前的時間
##### 指令
利用 date 的 --date 參數指定 150 天前，並指定輸出的時間格式。
```bash
[root@Server]# date +%Y%m%d --date="-150 day"
```

## Tip9	查詢伺服器serial number
##### 參考資料
[dmidecode](http://www.nongnu.org/dmidecode/)
##### 需求
查詢硬體資料(serial number)
##### 指令
利用 date 的 -t 參數指定查詢system資訊。
```bash
[root@Server]# dmidecode -t system
```

## Tip10 在shell中輸入tab
##### 需求
使用cut指令在分隔符號中輸入\t無效
##### 指令
先輸入Ctrl+V再輸入tab
```bash
[root@Server]# cut -d ' ' -f1,2,3 test.txt
```

## Tip11 比較兩文件的交集、差集及聯集
##### 需求
複製大量檔案過程，希望先刪除已複製過的檔案。
```bash
[root@Server]# cat orig.txt
1.csv
2.csv
3.csv
4.csv
5.csv
[root@Server]# cat copy.txt
1.csv
2.csv
3.csv
```
上述可知原本檔案有5個，目前已經複製3個檔案
##### 指令
uniq參數為u代表僅出現一次的行；參數為d代表僅出現重複的行。
```bash
# 交集
[root@Server]# sort orig.txt copy.txt|uniq -d
# 差集(orig.txt-copy.txt)
[root@Server]# sort orig.txt copy.txt|uniq -u
# 聯集
[root@Server]# sort orig.txt copy.txt|uniq
```
上述檔案內容為orig.txt包含copy.txt內容，但是當orig1.txt各自有不同內容copy1.txt，如何找出orig1.txt - copy1.txt的差集？
```bash
[root@Server]# cat orig1.txt
1.csv
2.csv
3.csv
4.csv
5.csv
[root@Server]# cat copy1.txt
a.csv
b.csv
3.csv
[root@Server]# sort orig1.txt copy1.txt copy1.txt|uniq -u
1.csv
2.csv
```

## Tip12 間接計算CSV欄位個數
##### 需求
計算CSV檔案中的欄位個數
CSV檔案內容如下：
```
A,B,C,D,E
F,G,H,I,J
K,L,M,N,O
```

##### 指令
1. 利用head指令印出1行內容
2. 利用tr指令接dc參數代表只萃取逗號
3. 利用wc指令接c參數計算有多少字元

利用這方法計算每行有4個逗號，可以推測出有5個欄位。
```bash
[root@Server]# head -n1 example.csv|tr -dc ','|wc -c
4
```

## Tip13  修改top指令設定檔
##### 需求
監控伺服器設備CPU與Mem使用率過程，希望透過top指令觀看所有CPU使用狀況。

##### 指令
1. 使用top指令觀察目前伺服器運作情形
2. 按1進入互動模式觀看各CPU使用狀況
3. 按大寫W將目前設定寫入設定檔

未來使用top指令預設會顯示各CPU使用狀況

## Tip14 透過rsync傳送近期生成檔案後並刪除
##### 需求
1. SFTP伺服器固定收容分析報告
2. SFTP伺服器硬碟空間不足，僅能存取一週分析報告。
3. 希望固定時間傳送報告後並刪除報告

##### 指令
1. 利用avRt參數保持所有文件屬性與時間
2. 利用find指令讀取特定目錄資料，並且存取時間為一天內的檔案，後續將檔案清單以rsync --files-from參數讀取。
3. 利用--remove-source-files參數設定檔案傳輸後刪除
4. 利用--append參數設定當目的端有同名但較小的檔案時，附加檔案內容。
5. /home/read_dir為rsync讀取的目錄
6. root@[Remote IP]:/home/receive_dir為傳送到遠端IP位址與指定目錄位置

```bash
[root@Server]# rsync -avRt --files-from=<(find [0-9]*-[0-9]*-[0-9]* -type f -atime -1) --remove-source-files --append /home/read_dir root@[Remote IP]:/home/receive_dir
```

如要固定時間執行rsync傳輸檔案，需設定以下步驟。
1. 複製SSH public key至遠端IP位址(hint: ssh-copy-id)
2. 將指令寫入crontab中
