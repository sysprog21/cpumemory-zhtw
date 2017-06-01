# A.1 矩陣乘法

這是在 6.2.1 節的矩陣乘法的完整基準測試程式。有關使用的內建函數的細節，請讀者參閱 Intel 的參考手冊。

```c
#include <stdlib.h>
#include <stdio.h>
#include <emmintrin.h>
#define N 1000
double res[N][N] __attribute__ ((aligned (64)));
double mul1[N][N] __attribute__ ((aligned (64)));
double mul2[N][N] __attribute__ ((aligned (64)));
#define SM (CLS / sizeof (double))

int
main (void)
{
  // ... Initialize mul1 and mul2
  int i, i2, j, j2, k, k2;
  double *restrict rres;
  double *restrict rmul1;
  double *restrict rmul2;
  for (i = 0; i < N; i += SM)
    for (j = 0; j < N; j += SM)
      for (k = 0; k < N; k += SM)
        for (i2 = 0, rres = &res[i][j], rmul1 = &mul1[i][k]; i2 < SM;
             ++i2, rres += N, rmul1 += N)
          {
            _mm_prefetch (&rmul1[8], _MM_HINT_NTA);
            for (k2 = 0, rmul2 = &mul2[k][j]; k2 < SM; ++k2, rmul2 += N)
              {
                __m128d m1d = _mm_load_sd (&rmul1[k2]);
                m1d = _mm_unpacklo_pd (m1d, m1d);
                for (j2 = 0; j2 < SM; j2 += 2)
                  {
                    __m128d m2 = _mm_load_pd (&rmul2[j2]);
                    __m128d r2 = _mm_load_pd (&rres[j2]);
                    _mm_store_pd (&rres[j2],
                                  _mm_add_pd (_mm_mul_pd (m2, m1d), r2));
                  }
              }
          }
  // ... use res matrix
  return 0;
}
```

迴圈的結構跟 6.2.1 節的最終型態幾乎完全相同。唯一的大改變是 `rmul1[k2]` 值的載入被拉出內部迴圈了，因為我們必須建立一個擁有兩個相同元素值的向量。這即是 `_mm_unpacklo_pd()` 內建函數所做的。

其餘唯一值得注意的事情是，我們明確地對齊了三個陣列，以令我們預期會位在同個快取行的值真的在那裡找到。

