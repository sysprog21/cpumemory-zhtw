# 3.5.4. FSB 的影響

<figure>
  <img src="../../assets/figure-3.32.png" alt="圖 3.32：FSB 速度的影響">
  <figcaption>圖 3.32：FSB 速度的影響</figcaption>
</figure>

FSB 在機器的效能中扮演一個重要的角色。快取內容只能以跟記憶體的連線所允許的一樣快地被儲存與寫入。我們能夠藉由在兩台僅在記憶體模組速度上有差異的機器上執行一支程式，來看看到底怎麼樣。圖 3.32 顯示了以一台 64 位元機器、`NPAD`=7 而言，Addnext0 測試（將下一個元素的 `pad[0]` 元素加到自己的 `pad[0]` 元素上）的結果。兩台機器都擁有 Intel Core 2 處理器，第一台使用 667MHz DDR2 模組，第二台則是 800MHz 模組（提升了 20%）。

數據顯示，當 FSB 真的受很大的工作集大小所壓迫時，我們的確看到了巨大的優勢。在這項測試中，量測到的最大效能提升為 18.2%，接近理論最大值。這表示，更快的 FSB 確實能夠省下大量的時間。當工作集大小能塞入快取時（這些處理器有一個 4MB L2），這並不重要。必須記在心上的是，這裡我們測量的是一支程式。一個系統的工作集包含所有同時執行的行程所需的記憶體。如此一來，以小得多的程式就可能輕易超過 4MB 以上的記憶體。

現今，一些 Intel 的處理器支援加速到 1,333MHz 的 FSB，這可能代表著額外 60% 的提升。未來將會看到更高的速度。若是速度很重要、並且工作集大小更大了，肯定是值得投資金錢在快速的 RAM 與很高的 FSB 速度的。不過必須小心，因為即使處理器可能會支援更高的 FSB 速度，但主機板／北橋可能不會。檢查規格是至關重要的。
