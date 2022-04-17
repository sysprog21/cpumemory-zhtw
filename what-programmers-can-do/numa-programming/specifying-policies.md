# 6.5.2. 指定策略

`set_mempolicy` 呼叫能夠用以為目前的執行緒（對系統核心來說的任務）設定任務策略。僅有目前的執行緒會受影響，而非整個行程。

```c
#include <numaif.h>
long set_mempolicy(int mode,
                   unsigned long *nodemask,
                   unsigned long maxnode);
```

`mode` 參數必須為前一節介紹過的其中一個 `MPOL_*` 常數。`nodemask` 參數指定未來分配要使用的記憶體節點，而 `maxnode` 為 `nodemask` 中的節點（即位元）數量。若是 `mode` 為 `MPOL_DEFAULT`，就不需要指定記憶體節點，並且會忽略 `nodemask` 參數。若是為 `MPOL_PREFERRED` 傳遞一個空指標作為 `nodemask`，則會選擇本地節點。否則，`MPOL_PREFERRED` 會使用 `nodemask` 中設置的位元所對應的最低的節點編號。

對於已經分配的記憶體，設定策略並沒有任何影響。分頁不會被自動遷移；只有未來的分配會受影響。注意到記憶體分配與預留定址空間之間的不同：一個使用 `mmap` 建立的定址空間區域通常不會被自動分配。在記憶體區域上首次的讀取或寫入操作會分配合適的分頁。若是策略在存取相同定址空間區域的不同分頁之間發生改變，或者策略允許記憶體的分配來自不同節點，那麼一個看似均勻的定址空間區域可能會被分散在許多記憶體節點之中。

