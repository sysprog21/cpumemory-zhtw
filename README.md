# 每位程式開發者都該知道的記憶體知識

本文翻譯自 [Ulrich Drepper](https://de.wikipedia.org/wiki/Ulrich_Drepper) 於 2007 年撰寫的論文《[What Every Programmer Should Know About Memory](https://www.akkadia.org/drepper/cpumemory.pdf)》(版次: 1.0)，原文共 114 頁。

在 CPU 核 (core) 在速度和數量增長的同時，記憶體存取限制著當今大多數程式的效率，甚至未來一段時間也會如此。
儘管硬體設計者已提出日趨複雜的記憶體處理與加速機制 —— 例如 CPU 快取 (cache) —— 但若程式開發者無法善用，仍無法有效發揮硬體作用。
不幸的是，論及電腦的記憶體子系統或 CPU 快取時，無論是其內部的結構，抑或存取成本，對大多程式開發者仍相當陌生。
本文解釋用於現代電腦硬體的記憶體子系統的結構、闡述 CPU 快取發展的考量、它們如何運作，以及程式該如何針對記憶體操作調整，從而達到最佳的效能。

## 翻譯資訊
譯者: [Chi-En Wu](https://github.com/jason2506)
