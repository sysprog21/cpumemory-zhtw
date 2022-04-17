# 6.4.2. 最小操作的最佳化

假如多條執行緒同時修改相同的記憶體位置，處理器並不保證任何具體的結果。這是個為了避免在所有情況的 99.999% 中的不必要成本而做出的慎重決定。舉例來說，若有個在「S」狀態的記憶體位置、並且有兩條執行緒同時必須增加它的值的時候，在從快取讀出舊值以執行加法之前，執行管線不必等待快取行變為「E」狀態。而是會讀取目前快取中的值，並且一旦快取行變為「E」狀態，新的值便會被寫回去。若是在兩條執行緒中的兩次快取讀取同時發生，結果並不如預期；其中一個加法會沒有效果。

對於可能發生並行操作的情況，處理器提供 atomic 操作。舉例來說，這些 atomic 操作可能在直到能以像是 atomic 地對記憶體位置進行加法的方式執行加法之前，不會讀取舊值。除了等待其它處理器核之外，某些處理器甚至會將特定位址的 atomic 操作發給在主機板上的其它裝置。這全都會令 atomic 操作變慢。

處理器廠商決定提供不同的一組 atomic 操作。早期的 RISC 處理器，與代表簡化（reduced）的「R」相符，提供非常少的 atomic 操作，有時僅有一個 atomic 的位元設置與測試。[^40]在光譜的另一端，我們有提供大量 atomic 操作的 x86 與 x86-64。普遍來說可用的 atomic 操作能夠歸納成四類：

<dl>
  <dt>位元測試</dt>
  <dd>這些操作 atomic 地設置或者清除一個位元，並回傳一個代表位元先前是否被設置的狀態。</dd>

  <dt>載入鎖定／條件儲存（Load Lock/Store Conditional，LL/SC）[^41]</dt>
  <dd>LL/SC 操作成對使用，其中特殊的載入指令用以開始一個事務（transaction），而最後的儲存僅會在這個位置沒有在這段期間內被修改的情況才會成功。儲存操作指出成功或失敗，所以程式能夠在必要時重複它的工作。</dd>

  <dt>比較並交換（Compare-and-Swap，CAS）</dt>
  <dd>這是個三元（ternary）操作，僅在目前值與第三個參數值相同的時候，將一個以參數提供的值寫入到一個位址中（第二個參數）；</dd>

  <dt> atomic 算術</dt>
  <dd>這些操作僅在 x86 與 x86-64 可用，其能夠在記憶體位置上執行算術與邏輯操作。這些處理器擁有對這些操作的非 atomic 版本的支援，但 RISC 架構則否。所以，怪不得它們的可用性是有限的。</dd>
</dl>

一個處理器架構可能會支援 LL/SC 指令或 CAS 指令其一，不會兩者都支援。兩種方法基本上相同；它們能提供一樣好的 atomic 算術操作實作，但看起來 CAS 是近來偏好的方法。其它所有的操作都能夠間接地以它來實作。例如，一個 atomic 加法：

```c
int curval;
int newval;
do {
  curval = var;
  newval = curval + addend;
} while (CAS(&var, curval, newval));
```

呼叫 `CAS` 的結果指出操作是否成功。若是它回傳失敗（非零的值），迴圈會再次執行、執行加法、並且再次嘗試呼叫 `CAS`。這會重複到成功為止。這段程式值得注意的是，記憶體位置的位址必須以兩個獨立的指令來計算。[^42]對於 LL/SC，程式看起來大致相同：

```c
int curval;
int newval;
do {
  curval = LL(var);
  newval = curval + addend;
} while (SC(var, newval));
```

這裡我們必須使用一個特殊的載入指令（`LL`），而且我們不必將記憶體位置的目前值傳遞給 `SC`，因為處理器知道記憶體位置是否曾在這期間被修改過。

<figure>
  <table>
    <tr>
      <td><pre><code>for (i = 0; i < N; ++i)
  __sync_add_and_fetch(&var,1);</code></pre></td>
      <td><pre><code>for (i = 0; i < N; ++i)
  __sync_fetch_and_add(&var,1);</code></pre></td>
      <td><pre><code>for (i = 0; i < N; ++i) {
  long v, n;
  do {
    v = var;
    n = v + 1;
  } while (!__sync_bool_compare_and_swap(&var, v,n));
}</code></pre></td>
    </tr>
    <tr>
      <th>1. 做加法並讀取結果</th>
      <th>2. 做加法並回傳舊值</th>
      <th>3.  atomic 地以新值替換</th>
    </tr>
  </table>
  <figcaption>圖 6.12：在一個迴圈中 atomic 遞增</figcaption>
</figure>

The big differentiators are x86 and x86-64, where we have the atomic operations and, here, it is important to select the proper atomic operation to achieve the best result.
圖 6.12 顯示實作一個 atomic 遞增操作的三種方法。在 x86 與 x86-64 上，三種方法全都會產生不同的程式，而在其它的架構上，程式則可能完全相同。效能的差異很大。下面的表格顯示由四條並行的執行緒進行 1 百萬次遞增的執行時間。程式使用 gcc 的內建函式（`__sync_*`）

<table>
  <tr>
    <th>1. Exchange Add</th>
    <th>2. Add Fetch</th>
    <th>3. CAS</th>
  </tr>
  <tr>
    <td>0.23s</td>
    <td>0.21s</td>
    <td style="background: yellow">0.73s</td>
  </tr>
</table>

前兩個數字很相近；我們看到回傳舊值稍微快一點。重要的資訊在被強調的那一欄，使用 CAS 時的成本。毫不意外，它要昂貴許多。對此有諸多理由：1. 有兩個記憶體操作、2. CAS 操作本身比較複雜，甚至需要條件操作、以及 3. 整個操作必須在一個迴圈中完成，以防兩個同時的存取造成一次 CAS 呼叫失敗。

現在讀者可能會問個問題：為什麼有人會使用這種利用 CAS 的複雜、而且較長的程式？對此的回答是：複雜性通常會被隱藏。如同先前提過的，CAS 是橫跨所有有趣架構的統一 atomic 操作。所以有些人認為，以 CAS 定義所有的 atomic 操作就足夠。這令程式更為簡單。但就如數字所顯示的，這絕對不是最好的結果。CAS 解法的記憶體管理的間接成本很大。下面示意僅有兩條執行緒的執行，每條在它們自己的處理器核上。

<table>
  <tr>
    <th>執行緒 #1</th>
    <th>執行緒 #2</th>
    <th><code>var</code> 快取狀態</th>
  </tr>
  <tr>
    <td><code>v = var</code></td>
    <td></td>
    <td>在 Proc 1 上為「E」</td>
  </tr>
  <tr>
    <td><code>n = v + 1</code></td>
    <td><code>v = var</code></td>
    <td>在 Proc 1+2 上為「S」</td>
  </tr>
  <tr>
    <td>CAS(<code>var</code>)</td>
    <td><code>n = v + 1</code></td>
    <td>在 Proc 1 上為「E」</td>
  </tr>
  <tr>
    <td></td>
    <td>CAS(<code>var</code>)</td>
    <td>在 Proc 2 上為「E」</td>
  </tr>
</table>

我們看到，在這段很短的執行期間內，快取行狀態至少改變三次；兩次改變為 RFO。再加上，因為第二個 CAS 會失敗，所以這條執行緒必須重複整個操作。在這個操作的期間，相同的情況可能會再度發生。

相比之下，在使用 atomic 算術操作時，處理器能夠將執行加法（或者其它什麼的）所需的載入與儲存操作保持在一起。能夠確保同時發出的快取行請求直到 atomic 操作完成前都會被阻擋。

因此，在範例中的每次迴圈迭代至多會產生一次 RFO 快取請求，就沒有別的。

這所有的一切都意味著，在一個能夠使用 atomic 算術與邏輯操作的層級定義機器抽象是很重要的。CAS 不該被普遍地用作統一的機制。

對於大多數處理器而言， atomic 操作本身一直是 atomic。對於不需要 atomic 的情況，只有藉由提供完全獨立的程式路徑時，才能夠避免這點。
This means more code, a conditional, and further jumps to direct execution appropriately.

對於 x86 與 x86-64，情況就不同：相同的指令能夠以 atomic 與非 atomic 的方式使用。為了令它們 atomic 化，便對指令用上一個特殊的前綴：`lock` 前綴。假如在一個給定的情況下， atomic 需求是不必要的，這為 atomic 操作提供避免高成本的機會。例如，在函式庫中，在需要時必須一直是執行緒安全（thread-safe）的程式就能夠受益於此。沒有撰寫程式時所需的資訊，決策能夠在執行期進行。技巧是跳過 `lock` 前綴。這個技巧適用於 x86 與 x86-64 允許以 `lock` 前綴的所有指令。

```
    cmpl $0, multiple_threads
    je   1f
    lock
1:  add  $1, some_var
```

如果這段組合語言程式看起來很神秘，別擔心，它很簡單的。第一個指令檢查一個變數是否為零。非零在這個情況中表示有多於一條執行中的執行緒。若是這個值為零，第二個指令就會跳到標籤 `1`。否則，就執行下一個指令。這就是狡猾的部分。若是 `je` 沒有跳躍，`add` 指令便會以 `lock` 前綴執行。否則，它會在沒有 `lock` 前綴的情況下執行。

增加像是條件跳躍這樣一個潛在昂貴的操作（在分支預測錯誤的情況下是很昂貴的）看似事與願違。確實可能如此：若是大多時候都有多條執行緒在執行中，效能會進一步降低，尤其在分支預測不正確的情況。但若是有許多僅有一條執行緒在使用中的情況，程式是明顯比較快的。使用一個 if-then-else 構造的替代方法在兩種情況下都會引入額外的非條件跳躍，這可能更慢。假定一次 atomic 操作花費大約 200 個週期，使用這個技巧（或是 if-then-else 區塊）的交叉點是相當低的。這肯定是個要記在心上的技術。不幸的是，這表示無法使用 gcc 的 `__sync_*` 內建函式。



[^40]: HP Parisc 仍然沒有提供更多的操作...

[^41]: 有些人會使用「鏈結（linked）」而非「鎖定」，這是一樣的。

[^42]: x86 與 x86-64 上的 `CAS` 操作碼（opcode）能夠避免第二次與後續迭代中的值的載入，但在這個平台上，我們能夠用一個單一的加法操作碼、以一個較簡單的方式來撰寫 atomic 加法。

