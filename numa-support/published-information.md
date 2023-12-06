# 5.3. 被發布的資訊

系統核心透過 `sys` 虛擬檔案系統（sysfs）將處理器快取的資訊發布在

`/sys/devices/system/cpu/cpu*/cache`

在 6.2.1 節，我們會看到能用來查詢不同快取容量的介面。這裡重要的是快取的拓樸。上面的目錄包含列出 CPU 擁有的不同快取資訊的子目錄（叫做 `index*`）。檔案 `type`、`level`、與 `shared_cpu_map` 是在這些目錄中與拓樸有關的重要檔案。一個 Intel Core 2 QX6700 的資訊看起來就如表 5.1。

<figure>
  <table>
    <tr>
      <th colspan="2"></th>
      <th><code>type</code></th>
      <th><code>level</code></th>
      <th><code>shared_cpu_map</code></th>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu0</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000003</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu1</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000003</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu2</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>0000000c</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu3</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>0000000c</td>
    </tr>
  </table>
  <figcaption>表 5.1：Core 2 CPU 快取的 <code>sysfs</code> 資訊</figcaption>
</figure>

這份資料的意義如下：

* 每顆處理器核[^25]擁有三個快取：L1i、L1d、L2。
* L1d 與 L1i 快取沒有被任何其它的處理器核所共享 –– 每顆處理器核有它自己的一組快取。這是由 `shared_cpu_map` 中的位元圖（bitmap）只有一個被設置的位元所暗示的。
* `cpu0` 與 `cpu1` 的 L2 快取是共享的，正如 `cpu2` 與 `cpu3` 上的 L2 一樣。

若是 CPU 有更多快取階層，也會有更多的 `index*` 目錄。

<figure>
  <table>
    <tr>
      <th colspan="2"></th>
      <th><code>type</code></th>
      <th><code>level</code></th>
      <th><code>shared_cpu_map</code></th>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu0</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu1</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu2</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu3</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu4</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu5</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu6</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td rowspan="3"><code>cpu7</code></td>
      <td><code>index0</code></td>
      <td>Data</td>
      <td>1</td>
      <td>00000080</td>
    </tr>
    <tr>
      <td><code>index1</code></td>
      <td>Instruction</td>
      <td>1</td>
      <td>00000080</td>
    </tr>
    <tr>
      <td><code>index2</code></td>
      <td>Unified</td>
      <td>2</td>
      <td>00000080</td>
    </tr>
  </table>
  <figcaption>表 5.2：Opteron CPU 快取的 <code>sysfs</code> 資訊</figcaption>
</figure>

對於一個四槽、雙核的 Opteron 機器，快取資訊看起來如表 5.2。可以看出這些處理器也有三種快取：L1i、L1d、L2。沒有處理器核共享任何階層的快取。這個系統有趣的部分在於處理器拓樸。少了這個額外資訊，就無法理解快取資料。`sys` 檔案系統將這個資訊擺在下面這個檔案

`/sys/devices/system/cpu/cpu*/topology`

表 5.3 顯示了在 SMP Opteron 機器的這個階層裡頭的令人感興趣的檔案。

<figure>
  <table>
    <tr>
      <th></th>
      <th><code>physical_<br />package_id</code></th>
      <th><code>core_id</code></th>
      <th><code>core_<br />siblings</code></th>
      <th><code>thread_<br />siblings</code></th>
    </tr>
    <tr>
      <td><code>cpu0</code></td>
      <td rowspan="2">0</td>
      <td>0</td>
      <td>00000003</td>
      <td>00000001</td>
    </tr>
    <tr>
      <td><code>cpu1</code></td>
      <td>1</td>
      <td>00000003</td>
      <td>00000002</td>
    </tr>
    <tr>
      <td><code>cpu2</code></td>
      <td rowspan="2">1</td>
      <td>0</td>
      <td>0000000c</td>
      <td>00000004</td>
    </tr>
    <tr>
      <td><code>cpu3</code></td>
      <td>1</td>
      <td>0000000c</td>
      <td>00000008</td>
    </tr>
    <tr>
      <td><code>cpu4</code></td>
      <td rowspan="2">2</td>
      <td>0</td>
      <td>00000030</td>
      <td>00000010</td>
    </tr>
    <tr>
      <td><code>cpu5</code></td>
      <td>1</td>
      <td>00000030</td>
      <td>00000020</td>
    </tr>
    <tr>
      <td><code>cpu6</code></td>
      <td rowspan="2">3</td>
      <td>0</td>
      <td>000000c0</td>
      <td>00000040</td>
    </tr>
    <tr>
      <td><code>cpu7</code></td>
      <td>1</td>
      <td>000000c0</td>
      <td>00000080</td>
    </tr>
  </table>
  <figcaption>表 5.3：Opteron CPU 拓樸的 <code>sysfs</code> 資訊</figcaption>
</figure>

將表 5.2 與 5.3 擺在一起，我們能夠發現

* 沒有 CPU 擁有 HT （`thethread_siblings` 位元圖有一個位元被設置）、
* 這個系統實際上共有四個處理器（`physical_package_id` 0 到 3）、
* 每個處理器有二顆核、以及
* 沒有處理器核共享任何快取。

這正好與較早期的 Opteron 一致。

目前為止提供的資料中完全缺少的是，有關這台機器上的 NUMA 性質的資訊。任何 SMP Opteron 機器都是一台 NUMA 機器。為了這份資料，我們必須看看在 NUMA 機器上存在的 `sys` 檔案系統的另一個部分，即下面的階層中

`/sys/devices/system/node`

這個目錄包含在系統上的每個 NUMA 節點的子目錄。在特定節點的目錄中有許多檔案。在前二張表中描述的 Opteron 機器的重要檔案與它們的內容顯示在表 5.4。

<figure>
  <table>
    <tr>
      <th></th>
      <th><code>cpumap</code></th>
      <th><code>distance</code></th>
    </tr>
    <tr>
      <td><code>node0</code></td>
      <td>00000003</td>
      <td>10 20 20 20</td>
    </tr>
    <tr>
      <td><code>node0</code></td>
      <td>0000000c</td>
      <td>20 10 20 20</td>
    </tr>
    <tr>
      <td><code>node2</code></td>
      <td>00000030</td>
      <td>20 20 10 20</td>
    </tr>
    <tr>
      <td><code>node3</code></td>
      <td>000000c0</td>
      <td>20 20 20 10</td>
    </tr>
  </table>
  <figcaption>表 5.4：Opteron 節點的 <code>sysfs</code> 資訊</figcaption>
</figure>

這個資訊將所有的一切連繫在一起；現在我們有個機器架構的完整輪廓了。我們已經知道機器擁有四個處理器。每個處理器構成它自己的節點，能夠從 `node*` 目錄的 `cpumap` 檔案中的值裡頭設置的位元看出來。在那些目錄的 `distance` 檔案包含一組值，一個值代表一個節點，表示在各個節點上存取記憶體的成本。在這個例子中，所有本地記憶體存取的成本為 10，所有對任何其它節點的遠端存取的成本為 20。[^26]這表示，即使處理器被組織成一個二維超立方體（見圖 5.1），在沒有直接連接的處理器之間存取也沒有比較貴。成本的相對值應該能用來作為存取時間的實際差距的估計。所有這些資訊的準確性是另一個問題了。


[^25]: `cpu0` 到 `cpu3` 為處理器核的相關資訊來自於另一個將會簡短介紹的地方。

[^26]: 順帶一提，這是不正確的。這個 ACPI 資訊明顯是錯的，因為 –– 雖然用到的處理器擁有三條連貫的超傳輸連結 –– 至少一個處理器必須被連接到一個南橋上。至少一對節點必須因此有比較大的距離。

