# linux_own-wait-queue
# 新增wait queue以及將process放入或移出wait queue

## 簡介
這次專案總共有四個檔，其中兩個為system call，另外兩個則為驗證system call是否正確的程式
以下介紹各個檔案的功能及細節
## sleep_process.c
定義了我們自己創建的三個wait queue
分別是project2_queue_1, project2_queue_2, project2_queue_3
其中使用的macro為[DECLARE_WAIT_QUEUE_HEAD](https://elixir.bootlin.com/linux/v3.19.1/source/include/linux/wait.h#L63)
```
DECLARE_WAIT_QUEUE_HEAD(project2_queue_1)
```

DECLARE_WAIT_QUEUE_HEAD為靜態(static)定義，也就是不會隨著system call結束就釋放分配給它的空間
且它會分配一個wait_queue_head_t的空間，並呼叫__WAIT_QUEUE_HEAD_INITIALIZER

linux有提供另一個funtion可創建wait queue

```
__init_waitqueue_head
```
但是該[function](https://elixir.bootlin.com/linux/v3.19.1/source/kernel/sched/wait.c#L14)創建的為動態分布，故在本專案中不使用

```
bool flag1 = false;
```
該flag為設定各個wake queue喚醒process的條件
在下面的code中
```
wait_event_interruptible(project2_queue_1,flag1 == true);
```
第二項參數所表示的即為喚醒條件
且使用[wait_event_interruptible](https://elixir.bootlin.com/linux/v3.19.1/source/include/linux/wait.h#L390)把process設為可以被signal打斷的狀態
更方便我們之後做測試
此system call的使用方式為傳遞1~3的參數給他後會把呼叫這個system call的process移至相應wait queue
並進入SLEEP的狀態

## wake_process.c
喚醒process的system call
其中使用了
```
wake_up_interruptible_all
```
這個[macro](https://elixir.bootlin.com/linux/v3.19.1/source/include/linux/wait.h#L173)
將wait_queue_head_t(就是wait queue的head)作為參數傳進去
他會呼叫[__wake_up](https://elixir.bootlin.com/linux/v3.19.1/source/kernel/sched/wait.c#L89)這個macro
並設定將interruptible且non-exclusive且滿足喚醒條件的process喚醒
該system call使用方法為傳入1~3的參數決定要喚醒哪個wait queue的process
所以不能由在wait queue中的process呼叫，需要由其他process來喚醒

