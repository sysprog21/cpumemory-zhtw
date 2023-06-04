# 每位程式開發者都該有的記憶體知識

本文翻譯自 [Ulrich Drepper](https://de.wikipedia.org/wiki/Ulrich_Drepper) 於 2007 年撰寫的論文〈[What Every Programmer Should Know About Memory](https://www.akkadia.org/drepper/cpumemory.pdf)〉(版次: 1.0)，原文共 114 頁。

隨著 CPU 核 (core) 的速度和數量的增長，記憶體存取成為制約當今大多數程式效率的因素，且在未來一段時間內仍然如此。
儘管硬體設計者提越來越複雜的記憶體處理和加速機制，例如 CPU 快取，但若程式開發者無法善加利用，這些硬體機制仍無法發揮有效作用。
不幸的是，大多數程式開發者對於計算機的記憶體子系統或 CPU 快取，無論是其內部結構還是存取成本，仍然相當陌生。
本文旨在解釋現代電腦硬體中記憶體子系統的結構，闡述 CPU 快取的發展考量，及它們的運作方式。
同時，本文也提供針對記憶體操作進行調整、達到最佳效能的建議。

## 翻譯資訊
譯者: [Chi-En Wu](https://github.com/jason2506), [Jim Huang](https://github.com/jserv)
> **[info]**
> 關於繁體中文翻譯內容的修正、改進建議，和貢獻，請造訪 [sysprog21/cpumemory-zhtw](https://github.com/sysprog21/cpumemory-zhtw)
