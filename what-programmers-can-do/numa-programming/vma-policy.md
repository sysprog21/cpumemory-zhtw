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

這個介面為位址範圍 [`start`, `start` + `len`) 註冊了一個新的 VMA 策略。由於記憶體管理是在分頁上操作，所以起始位址必須是對齊分頁的。`len` 值會被無條件進位至下一個分頁大小。

`mode` 參數再次指定了策略；這個值必須從 6.5.1 節的清單中挑選。與使用 `set_mempolicy` 相同，`nodemask` 參數只會用在某些策略上。它的處理是一樣的。

`mbind` 介面的語義取決於 `flags` 參數的值。預設情況下，若是 `flags` 為零，系統呼叫會為這個位址範圍設定 VMA 策略。現有的對映不受影響。假如這還不夠，目前有三種修改這種行為的旗標；它們能夠被單獨或者一起被選擇：

<dl>
    <dt><code>MPOL_MF_STRICT</code></dt>
    <dd>假如並非所有分頁都在由 <code>nodemask</code> 指定的節點上，對 <code>mbind</code> 的呼叫將會失敗。在這個旗標與 <code>MPOL_MF_MOVE</code> 和／或 <code>MPOL_MF_MOVEALL</code> 一起使用的情況下，呼叫會在任何分頁無法被移動的時候失敗。</dd>

    <dt><code>MPOL_MF_MOVE</code></dt>
    <dd>系統核心將會試著移動位址範圍中、任何分配在一個不在由 <code>nodemask</code> 指定的集合中的節點上的分頁。預設情況下，僅有被當前行程的分頁表專用的分頁會被移動。</dd>

    <dt><code>MPOL_MF_MOVEALL</code></dt>
    <dd>如同 <code>MPOL_MF_MOVE</code>，但系統核心會試著移動所有分頁，而非僅有那些獨自被當前行程的分頁表使用的分頁。這個操作具有系統層面的影響，因為它也影響了其它––可能不是由相同使用者所擁有的––行程的記憶體存取。因此 <code>MPOL_MF_MOVEALL</code> 是個特權操作（需要 <code>CAP_NICE</code> 的能力）。</dd>
</dl>

注意到針對 `MPOL_MF_MOVE` 與 `MPOL_MF_MOVEALL` 的支援僅在 2.6.16 Linux 系統核心中才被加入。

