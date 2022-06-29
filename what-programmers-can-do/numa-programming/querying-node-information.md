# 6.5.5. 查詢節點資訊

`get_mempolicy` 介面能用以查詢關於一個給定位址的 NUMA 狀態的各種事實。

```c
#include <numaif.h>
long get_mempolicy(int *policy,
             const unsigned long *nmask,
             unsigned long maxnode,
             void *addr, int flags);
```

當 `get_mempolicy` 以 `0` 作為 `flags` 參數呼叫時，關於位址 `addr` 的策略資訊會被儲存在由 `policy` 指到的字組、以及由 `nmask` 指到的節點的位元遮罩中。若是 `addr` 落在一段已經被指定一個 VMA 策略的定址空間範圍中，就回傳關於這個策略的資訊。否則，將會回傳關於任務策略或者––必要的話––系統預設策略的資訊。

若是 `flags` 中設定 `MPOL_F_NODE`、並且管理 `addr` 的策略為 `MPOL_INTERLEAVE`，那麼 `policy` 所指到的字組為要進行下一次分配的節點索引。這個資訊能夠潛在地用來設定打算要在新分配的記憶體上運作的一條執行緒的親和性。這可能是實現逼近的一個比較不昂貴的方法，尤其是在執行緒仍未被建立的情況。

`MPOL_F_ADDR` 旗標能用來檢索另一個完全不同的資料項目。假如使用這個旗標，`policy` 所指到的字組為已經為包含 `addr` 的分頁分配記憶體的記憶體節點索引。這個資訊能用來決定可能的分頁遷移、決定哪條執行緒能夠最有效率地運作在記憶體位置上、還有更多更多的事情。

一條執行緒正在使用的 CPU––以及因此用到的記憶體節點––比起它的記憶體分配還要更加變化無常。在沒有明確請求的情況下，記憶體分頁只會在極端的情況下被移動。一條執行緒能被指派給另一個 CPU，作為重新平衡 CPU 負載的結果。關於當前 CPU 與節點的資訊可能因而僅在短期內有效。排程器會試著將執行緒維持在同一個 CPU 上，甚至可能在相同的核上，以最小化由於冷快取（cold cache）造成的效能損失。這表示，查看當前 CPU 與節點的資訊是有用的；只要避免假設關聯性不會改變。

libNUMA 提供兩個介面，以查詢一段給定虛擬定址空間範圍的節點資訊：

```c
#include <libNUMA.h>
int NUMA_mem_get_node_idx(void *addr);
int NUMA_mem_get_node_mask(void *addr,
                           size_t size,
                           size_t __destsize,
                           memnode_set_t *dest);
```

`NUMA_mem_get_node_mask` 根據管理策略，在 `dest` 中設置代表所有分配（或者可能分配）範圍 [`addr`, `addr`+`size`) 中的分頁的記憶體節點的位元。`NUMA_mem_get_node` 只看位址 `addr`，並回傳分配（或者可能分配）這個位址的記憶體節點的索引。這些介面比 `get_mempolicy` 還容易使用，而且應該是首選。

當前正由一條執行緒使用的 CPU 能夠使用 `sched_getcpu` 來查詢（見 6.4.3 節）。使用這個資訊，一支程式能夠使用來自 libNUMA 的 `NUMA_cpu_to_memnode` 介面來確定 CPU 本地的記憶體節點（們）：

```c
#include <libNUMA.h>
int NUMA_cpu_to_memnode(size_t cpusetsize,
                        const cpu_set_t *cpuset,
                        size_t memnodesize,
                        memnode_set_t *
                        memnodeset);
```

對這個函式的一次呼叫會設置所有對應於任何在第二個參數指到的集合中的 CPU 本地的記憶體節點的位元。就如同 CPU 資訊本身，這個資訊直到機器的配置改變（例如，CPU 被移除或新增）時才會是正確的。

`memnode_set_t` 物件中的位元能被用在像 `get_mempolicy` 這種低階函式的呼叫上。使用 libNUMA 中的其它函式會更加方便。反向映射能透過下述函式取得：

```c
#include <libNUMA.h>
int NUMA_memnode_to_cpu(size_t memnodesize,
                        const memnode_set_t *
                        memnodeset,
                        size_t cpusetsize,
                        cpu_set_t *cpuset)
```

在產生的 `cpuset` 中設置的位元為任何在 `memnodeset` 中設置的位元所對應的記憶體節點本地的那些 CPU。對於這兩個介面，程式開發者都必須意識到，資訊可能隨著時間改變（尤其是使用 CPU 熱插拔的情況）。在許多情境中，在輸入的位元集中只會設置單一個位元，但舉例來說，將 `sched_getaffinity` 呼叫檢索到的整個 CPU 集合傳遞到 `NUMA_cpu_to_memnode`，以確定哪些記憶體節點能夠被執行緒直接存取到，也是有意義的。

