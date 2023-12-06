# 6.5.4. VMA 策略

要為一個位址範圍設定 VMA 策略，必須使用一個不同的介面：

```c
#include <numaif.h>
long mbind(void *start, unsigned long len,
           int mode,
           unsigned long *nodemask,
           unsigned long maxnode,
           unsigned flags);
```

這個介面為位址範圍 [`start`, `start` + `len`) 註冊一個新的 VMA 策略。由於記憶體管理是在分頁上操作，所以起始位址必須是對齊分頁的。`len` 值會被無條件進位至下一個分頁容量。

`mode` 參數再次指定策略；這個值必須從 6.5.1 節的清單中挑選。與使用 `set_mempolicy` 相同，`nodemask` 參數只會用在某些策略上。它的處理是一樣的。

`mbind` 介面的語義取決於 `flags` 參數的值。預設情況下，若是 `flags` 為零，系統呼叫會為這個位址範圍設定 VMA 策略。現有的對映不受影響。假如這還不夠，目前有三種修改這種行為的旗標；它們能夠被單獨或者一起被選擇：

<dl>
    <dt><code>MPOL_MF_STRICT</code></dt>
    <dd>假如並非所有分頁都在由 <code>nodemask</code> 指定的節點上，對 <code>mbind</code> 的呼叫將會失敗。在這個旗標與 <code>MPOL_MF_MOVE</code> 和／或 <code>MPOL_MF_MOVEALL</code> 一起使用的情況下，呼叫會在任何分頁無法被移動的時候失敗。</dd>

    <dt><code>MPOL_MF_MOVE</code></dt>
    <dd>系統核心將會試著移動位址範圍中、任何分配在一個不在由 <code>nodemask</code> 指定的集合中的節點上的分頁。預設情況下，僅有被目前行程的分頁表專用的分頁會被移動。</dd>

    <dt><code>MPOL_MF_MOVEALL</code></dt>
    <dd>如同 <code>MPOL_MF_MOVE</code>，但系統核心會試著移動所有分頁，而非僅有那些獨自被目前行程的分頁表使用的分頁。這個操作具有系統層面的影響，因為它也影響其它 –– 可能不是由相同使用者所擁有的 –– 行程的記憶體存取。因此 <code>MPOL_MF_MOVEALL</code> 是個特權操作（需要 <code>CAP_NICE</code> 的能力）。</dd>
</dl>

注意到針對 `MPOL_MF_MOVE` 與 `MPOL_MF_MOVEALL` 的支援僅在 2.6.16 Linux 系統核心中才被加入。

在沒有任何旗標的情況下呼叫 `mbind`，在任何分頁真的被分配之前必須為一個新預留的位址範圍指定策略的時候是最有用的。

```c
void *p = mmap(NULL, len,
               PROT_READ|PROT_WRITE,
               MAP_ANON, -1, 0);
if (p != MAP_FAILED)
  mbind(p, len, mode, nodemask, maxnode,
        0);
```

這段程式序列保留一段 `len` 位元組的定址空間範圍，並指定應該使用指涉 `nodemask` 中的記憶體節點的策略 `mode`。除非 `MAP_POPULATE` 旗標與 `mmap` 一起使用，否則在 `mbind` 呼叫的時候並不會分配任何記憶體，因此新的策略會套用在這個定址空間區域中的所有分頁。

單獨的 `MPOL_MF_STRICT` 旗標能用來確定，傳給 `mbind` 的 `start` 與 `len` 參數所描述的位址範圍中的任何分頁，是否都被分配在那些由 `nodemask` 指定的那些節點以外的節點上。已分配的分頁不會被改變。若是所有分頁都被分配在指定的節點上，那麼定址空間區域的 VMA 策略會根據 `mode` 改變。

有時候是需要記憶體的重新平衡的。在這種情況下，可能必須將一個節點上分配的分頁移到另一個節點上。以設置 `MPOL_MF_MOVE` 呼叫的 `mbind` 會盡最大努力來達成這點。僅有單獨被行程的分頁表樹指涉到的分頁會被考慮是否移動。可能有多個以執行緒或其他行程的形式共享部分分頁表樹的使用者。不可能影響碰巧映射相同資料的其它行程。這些分頁並不共享分頁表項目。

若是傳遞給 `mbind` 的 `flags` 參數中設置 `MPOL_MF_STRICT` 與 `MPOL_MF_MOVE` 位元，系統核心會試著移動並非分配在指定節點上的所有分頁。假如這無法做到，這個呼叫將會失敗。這種呼叫可能有助於確定是否有個節點（或是一組節點）能夠容納所有的分頁。可以連續嘗試多種組合，直到找到一個合適的節點。

除非執行目前的行程是這台電腦的主要目的，否則 `MPOL_MF_MOVEALL` 的使用是較難以證明為正當的。理由是，即使是出現在多張分頁表的分頁也會被移動。這會輕易地以負面的方式影響其它行程。因而應該要謹慎地使用這個操作。

