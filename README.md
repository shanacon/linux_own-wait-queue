# linux_own-wait-queue
# 新增wait queue以及將process放入或移出wait queue

## 簡介
這次專案總共有四個檔，其中兩個為system call，另外兩個則為驗證system call是否正確的程式
以下介紹各個檔案的功能及細節
###### sleep_process.c
定義了我們自己創建的三個wait queue
分別是project2_queue_1, project2_queue_2, project2_queue_3
其中使用的macro為DECLARE_WAIT_QUEUE_HEAD
```
DECLARE_WAIT_QUEUE_HEAD(project2_queue_1);
```
DECLARE_WAIT_QUEUE_HEAD為靜態(static)定義，也就是不會隨著system call結束就釋放分配給它的空間
且它會分配一個wait_queue_head_t的空間，並呼叫__WAIT_QUEUE_HEAD_INITIALIZER

linux有提供另一個funtion可創建wait queue
```
__init_waitqueue_head
```
但是該function創建的為動態分布，故在本專案中不使用

```
bool flag1 = false;
```
該flag為設定各個wake queue喚醒process的條件
在下面的code中
```
wait_event_interruptible(project2_queue_1,flag1 == true);
```
第二項參數所表示的即為喚醒條件
且使用wait_event_interruptible把process設為可以被signal打斷的狀態
更方便我們之後做測試
此system call的使用方式為傳遞1~3的參數給他後會把呼叫這個system call的process移至相應wait queue
並進入SLEEP的狀態

