# 6.2.4. 最佳化 TLB 使用

有兩種 TLB 使用的最佳化。第一種最佳化是減少一支程式必須使用的分頁數。這會自動導致較少的 TLB 錯失。第二種最佳化是藉由減少必須被分配的較高層目錄表的數量，以令 TLB 查詢便宜一些。較少的目錄表代表使用的記憶體較少，這可能使得目錄查詢有較高的快取命中率。

第一種最佳化與分頁錯誤的最小化密切相關。我們將會在 7.5 節仔細地涵蓋這個主題。分頁錯誤經常是個一次性的成本，但由於 TLB 快取通常很小而且會被頻繁地沖出，因此 TLB 錯失是個長期的損失。分頁錯誤比起 TLB 錯失還貴了數個數量級，但若是一支程式跑得足夠久、而且程式的某些部分會被足夠頻繁地執行，TLB 錯失甚至可能超過分頁錯誤的成本。因此重要的是，不僅要從分頁錯誤的角度、也要從 TLB 錯失的角度來考慮分頁最佳化。差異在於，分頁錯誤的最佳化只要求分頁範圍內的程式碼與資料分組，而 TLB 最佳化則要求––在任何時間點––盡可能少的 TLB 項目。

第二種 TLB 最佳化甚至更難控制。必須使用的分頁目錄數量是視行程的虛擬定址空間中使用的位址範圍分佈而定的。定址空間中廣泛多樣的位置代表著更多的目錄。

一個難題是，定址空間佈局隨機化（Address Space Layout Randomization，ASLR）恰好造成了這種狀況。堆疊、DSO、堆積、與可能的可執行檔的載入位址會在執行期隨機化，以防止機器的攻擊者猜出函數或變數的位址。

只有在最大效能至關重要的情況下，才應該關掉 ASLR。額外目錄的成本低到足以令這步是不必要的，除了一些極端的狀況之外。系統核心能隨時執行的一個可能的最佳化是，確保一個單一的映射不會橫跨兩個目錄之間的記憶體空間邊界。這會以最小的方式限制 ASLR，但不足以大幅地削弱它。

程式開發者直接受此影響的唯一方式是在明確請求一個定址空間區域的時候。這會在以 `MAP_FIXED` 使用 `mmap` 的時候發生。
Allocating new a address space region this way is very dangerous and hardly ever done. It is possible, though, and, if it is used and the addresses can be freely chosen, the programmer should know about the boundaries of the last level page directory and select the requested address appropriately.

