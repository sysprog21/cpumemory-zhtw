# 6.2. 快取存取

希望改進他們程式效能的程式開發者會發現，最好聚焦在影響一階快取的改變上，因為這很可能會產生最好的結果。我們將會在討論延伸到其它層級之前先討論它。顯然地，所有針對一階快取的最佳化也會影響其它快取。所有記憶體存取的主題都是相同的：改進局部性（空間與時間）並對齊程式碼與資料。

