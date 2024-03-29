# 6.3.3. 特殊的預取類型：猜測

一個現代處理器的 OoO 執行能力允許在不與彼此衝突的情況下搬移指令。舉例來說（這次使用 IA-64 為例）：

```
st8        [r4] = 12
add        r5 = r6, r7;;
st8        [r18] = r5
```

這段程式序列將 12 儲存至由暫存器 `r4` 指定的位址、將 `r6` 與 `r7` 暫存器的內容相加、並將它儲存在暫存器 `r5` 中。最後，它將總和儲存至由暫存器 `r18` 指定的位址。這裡的重點在於，加法指令能夠在第一個 `st8` 指令之前 –– 或者同時 –– 執行，因為並沒有資料的依賴關係。但假如必須載入其中一個加數會怎麼樣呢？

```
st8        [r4] = 12
ld8        r6 = [r8];;
add        r5 = r6, r7;;
st8        [r18] = r5
```

額外的 `ld8` 指令將值載入到由 `r8` 指令的位址。在這個載入指令與接下來的 `add` 指令之間有個明確的資料依賴關係（這便是指令後面的 `;;` 的理由，感謝提問）。這裡的關鍵在於，新的 `ld8` 指令 –– 不若 `add` 指令 –– 無法被移到第一個 `st8` 前面。處理器無法在指令解碼的期間足夠快速地決定儲存與載入是否衝突 –– 即，`r4` 與 `r8` 是否可能有相同的值。假如它們有相同的值，`st8` 指令會決定載入到 `r6` 的值。更糟的是，在載入錯失快取的情況下，`ld8` 可能也會隨之帶來漫長的等待時間。IA 64 架構針對這種情況支援猜測式載入（speculative load）：

```
ld8.a      r6 = [r8];;
[... other instructions ...]
st8        [r4] = 12
ld8.c.clr  r6 = [r8];;
add        r5 = r6, r7;;
st8        [r18] = r5
```

新的 `ld8.a` 與 `ld8.c.clr` 指令是一對的，並取代前一段程式序列的 `ld8` 指令。`ld8.a` 為猜測式載入。這個值無法被直接使用，但處理器能開始運作。這時，當到達 `ld8.c.clr` 指令的時候，這個內容可能已經被載入（假定這個間隔中有足夠數量的指令）。這個指令的引數（argument）必須與 `ld8.a` 指令相符。若是前面的 `st8` 指令沒有覆寫這個值（即 `r4` 與 `r8` 相同[^譯註]），就什麼也不必做。猜測式載入做它的工作，而載入的等待時間被隱藏。若是載入與儲存衝突，`ld8.c.clr` 會重新從記憶體載入值，而我們最終會得到一個正常的 `ld8` 指令的語義。

猜測式載入（仍？）沒有被廣泛使用。但如同這個例子所顯示的，它是個非常簡單而有效的隱藏等待時間的方法。預取基本上是等同的東西，並且對有著少量暫存器的處理器而言，猜測式載入可能沒多大意義。猜測式載入有直接將值載入到暫存器中，而不載入到可能會被再次逐出的快取行（舉例來說，當執行緒被移出排程〔deschedule〕的時候）這個（有時很大的）優點。如果能夠使用猜測的話，應該要使用它。



[^譯註]: `r4` 與 `r8` 相同指的是「值會被覆寫的情況」。

