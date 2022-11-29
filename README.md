[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Soliton-Analytics-Team/Heap_operator/blob/main/Heap_operator.ipynb)

# Colabで学ぶC++ヒープ演算と部分ソート完全攻略

make_heap(), push_heap(), pop_heap(), sort_heap()
これらの関数はC++のSTLに含まれているものですが、その有用性にもかかわらず、使い方に関する情報に乏しいです。最近、これらを深掘りしたので、知見を共有いたします。ただし、ヒープ構造に関する情報は散見されるため、ここではその点には触れず、その使い道や使い方についてColabで動く実例を交えながら解説します。

ヒープ演算は、Top Kの値の取得に使えます。Kが配列全体の長さと同じなら全ソート、短ければ部分ソートということになります。それもインプレイスで処理できるので、メモリリソースも最小限で済みます。単純にソートに使うものと思っている人が多いのですが、部分ソートに使えることを知っていると有用性の幅が広がります。

## 1. make_heap()の使い方

それではまず、基本として、make_heap()で何が生成されるのか見てみます。

```C++
%%writefile base.cpp

#include <algorithm>
#include <array>
#include <iostream>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::array<int, 6> a = {3, 1, 4, 2, 5, 6};
    std::make_heap(a.begin(), a.end());
    print("make_heap", a);
}
```

    Overwriting base.cpp

```shell
!g++ base.cpp
!./a.out
```

    make_heap 6 5 4 2 1 3

上記で分かるようにヒープ**構造**といっても、ただの数値の並びです。もちろん、一定の法則に沿って並べられているのですが、それについてはここでは述べません。ただ、最初の値a[0]は最大値となっていることは覚えておきましょう。

ところで、make_heap()の最小限の使い方は ```make_heap(a.begin(), a.end())``` で第一引数に開始位置、第二引数に終了位置を指定します。ということは、これらの位置をその他に指定すれば、配列の途中だけをヒープ構造にすることができるということです。例えば```make_heap(a.begin() + 1, a.begin() + 4)```とか。この場合はa[1] ,a[2], a[3]が並び替えられます。

## 2. sort_heap()の使い方

次にシンプルなsort_heap()を見てみましょう。

```C++
%%writefile sort.cpp

#include <algorithm>
#include <array>
#include <iostream>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::array<int, 6> a = {3, 1, 4, 2, 5, 6};
    std::make_heap(a.begin(), a.end());
    print("make_heap", a);
    std::sort_heap(a.begin(), a.end());
    print("sort_heap", a);
}
```

    Overwriting sort.cpp

```shell
!g++ sort.cpp
!./a.out
```

    make_heap 6 5 4 2 1 3
    sort_heap 1 2 3 4 5 6

小さい順に並びましたね。ヒープ構造の最初の値が最大値と聞いておや？と思った方もいらっしゃると思いますが、これをsort_heap()にかけると小さい順に並ぶのです。省略されている比較演算子はstd::less<>()です。これをstd::greater<>()に変更すると逆順になります。ちょっとやってみましょう。

```C++
%%writefile rev_sort.cpp

#include <algorithm>
#include <array>
#include <iostream>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::array<int, 6> a = {3, 1, 4, 2, 5, 6};
    std::make_heap(a.begin(), a.end(), std::greater<>());
    print("make_heap", a);
    std::sort_heap(a.begin(), a.end(), std::greater<>());
    print("sort_heap", a);
}
```

    Overwriting rev_sort.cpp

```shell
!g++ rev_sort.cpp
!./a.out
```

    make_heap 1 2 4 3 5 6
    sort_heap 6 5 4 3 2 1

大きい順になりましたね。もちろんこの場合、make_heap()で並べ替えた配列の最初の値は最小値になります。また、make_heap()とsort_heap()では同じ比較演算子を指定しなければなりません。

ちなみに、greater<>に<>が付いているのは、配列の要素の型に合わせて自動的に比較方法を調整する機能のためです。C++便利ですね。

ところで、vectorとかarrayじゃなくて、int[]みたいな配列を操作するにはどうすればいいでしょうか。

```C++
%%writefile sort_array.cpp

#include <algorithm>
#include <iostream>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    int a[] = {3, 1, 4, 2, 5, 6};
    std::make_heap(a, a + 6);
    print("make_heap", a);
    std::sort_heap(a, a + 6);
    print("sort_heap", a);
}
```

    Overwriting sort_array.cpp

```shell
!g++ sort_array.cpp
!./a.out
```

    make_heap 6 5 4 2 1 3
    sort_heap 1 2 3 4 5 6

簡単ですね！ただ配列の大きさを別途管理しなければなりません。

## 3. push_heap()の使い方

ソートするのに、make_heap()とsort_heap()の２段階に別れているのはなんのためでしょうか。これは配列を一括でソートするだけではなく、逐次的に値を追加できるようにするためです。その機能を担うのが、push_heap()です。

```C++
%%writefile push_sample.cpp

#include <algorithm>
#include <vector>
#include <iostream>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::vector<int> a = {3, 1, 4, 2, 5, 6};
    std::make_heap(a.begin(), a.end());
    print("make_heap", a);
    a.push_back(5);                     // ここで配列の末尾に値を追加
    print("push_back", a);
    std::push_heap(a.begin(), a.end()); // 追加した値も含めてpush_heap()
    print("push_heap", a);
    std::sort_heap(a.begin(), a.end());
    print("sort_heap", a);
}
```

    Overwriting push_sample.cpp

```shell
!g++ push_sample.cpp
!./a.out
```

    make_heap 6 5 4 2 1 3
    push_back 6 5 4 2 1 3 5
    push_heap 6 5 5 2 1 3 4
    sort_heap 1 2 3 4 5 5 6

既存のヒープ配列に要素を追加している様子が分かりますね。

## 4. pop_heap()の使い方

push_heap()で要素を追加できることは分かりましたが、全要素を追加するなら、最初から全要素に対してmake_heap()するだけで良いし、その方が効率もよいらしいので、あまりpush_heap()の有効性が分かりません。そこでpop_heap()の登場です。

pop_heap()は、先頭と末尾の要素を入れ替えた上で末尾を除いた部分についてヒープ構造を再構築します。ってなんのことでしょうか。意味がわかりませんね。ここで先頭の要素は最大値であることを思い出してください。先頭と末尾を入れ替えるということは、最大値を末尾に持ってくるということです。そして最大値を除いた残りでヒープ構造を構成するというわけです。その様子を以下で確認してみましょう。

```C++
%%writefile pop_sample.cpp

#include <algorithm>
#include <vector>
#include <iostream>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::vector<int> a = {3, 1, 4, 2, 5, 6};
    std::make_heap(a.begin(), a.end());
    print("make_heap", a);
    std::pop_heap(a.begin(), a.end());
    print("pop_heap", a);
}
```

    Overwriting pop_sample.cpp

```shell
!g++ pop_sample.cpp
!./a.out
```

    make_heap 6 5 4 2 1 3
    pop_heap 5 3 4 2 1 6

最大値が最後尾に来て、次の最大値が先頭に来ている様子が分かりますね。では何のためにこんなことをするのでしょうか。実は対象範囲を短くしながらこれを繰り返すと最終的に全ての要素が小さい順に並びます。結果、これがsort_heap()になるのですが、それは次で見ます。

と、これではpush_heap()の効用が説明できていません。実はpush_heap()とpop_heap()を繰り返すことで部分ソートが実現できるのです。それについては、partial_sort()の節で説明します。

## 5. sort_heap()の正体

pop_heap()を説明しましたので、sort_heap()の中身を理解できます。このアルゴリズムを知った時は驚愕しました。今までのヒープ演算の仕様は全てこのためであると言っても過言ではありません。本体は

```
  while (last - first > 1) {
    std::pop_heap(first, last, comp);
    --last;
  }
```

これだけです。あえて説明しませんが、うまく設計されているものですね。以下で実際に動かして検証します。

```C++
%%writefile my_sort_heap.cpp

#include <algorithm>
#include <array>
#include <iostream>

template <class RandomAccessIterator, class Compare>
void my_sort_heap(RandomAccessIterator first, RandomAccessIterator last, Compare comp) {
  while (last - first > 1) {
    std::pop_heap(first, last, comp);
    --last;
  }
}

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::array<int, 6> a = {3, 1, 4, 2, 5, 6};
    std::make_heap(a.begin(), a.end());
    print("make_heap", a);
    my_sort_heap(a.begin(), a.end(), std::less<>());
    print("my_sort_heap", a);
}
```

    Overwriting my_sort_heap.cpp

```shell
!g++ my_sort_heap.cpp
!./a.out
```

    make_heap 6 5 4 2 1 3
    my_sort_heap 1 2 3 4 5 6

std::sort_heap()と同様であることが確認できました。

## 6. partial_sort()

標準にpartial_sort()があるので、動作を確認します。

```C++
%%writefile partial_sort.cpp

#include <algorithm>
#include <vector>
#include <iostream>
#include <random>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::vector<int> a(20);
    std::iota(a.begin(), a.end(), 0);
    std::random_device seed_gen;
    std::mt19937 engine(seed_gen());
    std::shuffle(a.begin(), a.end(), engine);
    print("original", a);
    std::partial_sort(a.begin(), a.begin() + 5, a.end());
    print("partial_sort", a);
}
```

    Overwriting partial_sort.cpp

```shell
!g++ partial_sort.cpp
!./a.out
```

    original 1 7 14 13 16 3 12 15 2 6 10 8 11 5 19 0 18 9 4 17
    partial_sort 0 1 2 3 4 16 14 15 13 12 10 8 11 7 19 6 18 9 5 17

小さい順に5つの要素だけが配列の先頭に来ていることが分かります。全ソートのオーダーはO(N \* log N)ですが、部分ソートはO(N \* log K)ですので、高速化します。

それでは、push_heap()してからpop_heap()をした様子を見てみましょう。

```C++
%%writefile push_pop.cpp

#include <algorithm>
#include <vector>
#include <iostream>

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::vector<int> a = {3, 1, 4, 2, 6};
    print("original", a);
    std::make_heap(a.begin(), a.end());
    print("make_heap", a);
    a.push_back(5);
    print("push_back", a);
    std::push_heap(a.begin(), a.end());
    print("push_heap", a);
    std::pop_heap(a.begin(), a.end());
    print("pop_heap", a);
    a.pop_back();
    print("pop_back", a);
}
```

    Overwriting push_pop.cpp

```shell
!g++ push_pop.cpp
!./a.out
```

    original 3 1 4 2 6
    make_heap 6 3 4 2 1
    push_back 6 3 4 2 1 5
    push_heap 6 3 5 2 1 4
    pop_heap 5 3 4 2 1 6
    pop_back 5 3 4 2 1

5を追加投入すると、それまでの最大値だった6が除かれています。
これによって最小値から5番目までが常に保持される様子が分かると思います。

これを使って実装した部分ソートを試してみましょう。

```C++
%%writefile simple_partial_sort.cpp

#include <algorithm>
#include <vector>
#include <iostream>
#include <random>

template <class RandomAccessIterator, class Compare>
void simple_partial_sort(RandomAccessIterator first, RandomAccessIterator middle, RandomAccessIterator last, Compare comp) {
    std::make_heap(first, middle, comp);
    for (auto it = middle; it != last; ++it) {
        if (!comp(*it ,*first)) continue;
        std::iter_swap(middle, it);
        std::push_heap(first, middle + 1, comp);
        std::pop_heap(first, middle + 1, comp);
    }
    std::sort_heap(first, middle);
}

#define print(title, a) std::cout << title << "\t"; for (auto it : a) std::cout << it << " "; std::cout << std::endl;

int main(void) {
    std::vector<int> a(20);
    std::iota(a.begin(), a.end(), 0);
    std::random_device seed_gen;
    std::mt19937 engine(seed_gen());
    std::shuffle(a.begin(), a.end(), engine);
    print("original", a);
    simple_partial_sort(a.begin(), a.begin() + 5, a.end(), std::less<>());
    print("partial_sort", a);
}
```

    Overwriting simple_partial_sort.cpp

```shell
!g++ simple_partial_sort.cpp
!./a.out
```

    original 3 7 15 19 11 0 13 12 1 18 8 10 9 2 5 4 16 14 17 6
    partial_sort 0 1 2 3 4 5 19 15 13 18 12 10 9 11 8 7 16 14 17 6

実際に最小値から5番目までがソートされているのが分かります。

### 7. 処理時間比較

10回実行して処理時間とクロックを平均するマクロと、上位の結果を出力するマクロを定義します。

```C++
%%writefile util.h

#include <iostream>
#include <chrono>

#define N 10

#define T(title, f) {                                                       \
    float total_t = 0;                                                      \
    clock_t total_clock = 0;                                                \
    for (size_t i = i; i < N; ++i) {                                        \
      auto start = std::chrono::steady_clock::now();                        \
      clock_t start_clock = clock();                                        \
      f;                                                                    \
      clock_t end_clock = clock();                                          \
      auto end = std::chrono::steady_clock::now();                          \
      total_t +=                                                            \
        std::chrono::duration_cast<std::chrono::microseconds>(end - start).count() / 1000000.0; \
      total_clock |= end_clock - start_clock;                               \
    }                                                                       \
    std::cerr << title << "\tave: " << std::fixed << total_t / N << " sec " \
              << total_clock / N << " clock" << std::endl;                  \
  }

#define P(title, a, k) {                                        \
  std::cout << title << "\t";                                   \
  for (auto it = a; it != a + k; ++it) std::cout << *it << " "; \
  std::cout << std::endl; }

```

    Overwriting util.h

次に、上記の検討から得られたシンプルな部分ソートを定義します。

```C++
%%writefile simple_partial_sort.h

#include <algorithm>

template <class RandomAccessIterator, class Compare>
void simple_partial_sort(RandomAccessIterator first, RandomAccessIterator middle, RandomAccessIterator last, Compare comp) {
    std::make_heap(first, middle, comp);
    for (auto it = middle; it != last; ++it) {
        if (!comp(*it, *first)) continue;
        std::iter_swap(middle, it);
        std::push_heap(first, middle + 1, comp);
        std::pop_heap(first, middle + 1, comp);
    }
    std::sort_heap(first, middle);
}
```

    Overwriting simple_partial_sort.h

ただし、[このページ](https://en.cppreference.com/w/cpp/algorithm/partial_sort)によると、上記の実装は実践では若干効率が悪いそうです。push_heap()とpop_heap()を同時に行うsift_down()という関数を使う実装が載っていたので、それを拝借したのが下記のstd_partial_sort()です。
また、そこから最後のsort_heap()を除いたものをstd_nth_element()としてみました。std::nth_element()と同等のもので、topkを取得する関数ですが、得られた結果はソートされていません。

```C++
%%writefile std_partial_sort.h

#include <algorithm>

namespace impl {
template<typename RandomIt, typename Compare = std::less<typename std::iterator_traits<RandomIt>::value_type>>
void sift_down(RandomIt begin, RandomIt end, const Compare &comp = {}) { // sift down element at 'begin'
  const auto length = static_cast<size_t>(end - begin);
  size_t current = 0;
  size_t next = 2;
  while (next < length) {
    if (comp(*(begin + next), *(begin + (next - 1))))
      --next;
    if (!comp(*(begin + current), *(begin + next)))
      return;
    std::iter_swap(begin + current, begin + next);
    current = next;
    next = 2 * current + 2;
  }
  --next;
  if (next < length && comp(*(begin + current), *(begin + next)))
    std::iter_swap(begin + current, begin + next);
}

template<typename RandomIt, typename Compare = std::less<typename std::iterator_traits<RandomIt>::value_type>>
void heap_select(RandomIt begin, RandomIt middle, RandomIt end, const Compare &comp = {}) {
  std::make_heap(begin, middle, comp);
  for (auto i = middle; i != end; ++i)
    if (comp(*i, *begin)) {
      std::iter_swap(begin, i);
      sift_down(begin, middle, comp);
    }
}

} // namespace impl

template<typename RandomIt, typename Compare = std::less<typename std::iterator_traits<RandomIt>::value_type>>
void std_partial_sort(RandomIt begin, RandomIt middle, RandomIt end, Compare comp = {}) {
  impl::heap_select(begin, middle, end, comp);
  std::sort_heap(begin, middle, comp);
}

template<typename RandomIt, typename Compare = std::less<typename std::iterator_traits<RandomIt>::value_type>>
void std_nth_element(RandomIt begin, RandomIt middle, RandomIt end, Compare comp = {}) {
  impl::heap_select(begin, middle, end, comp);
}

```

    Overwriting std_partial_sort.h

本家のSTLのstd::nth_element()はkが小さい時は効率が悪いのですが、処理時間がkにあまり左右されない特徴があり、kが100万を超えてくると相対的に処理時間が短くなります。つまり、上記のstd_nth_element()のheapを使うのとは異なるアルゴリズムを採用しています。簡単に説明すると、配列を与えられた数値の大小で前後に分けるpartition()という線形オーダーの関数を使って、先頭に溜まる値がk個になるまで対象範囲を絞るのを繰り返します。前後に分けるための数値を適当に拾う、運任せのアルゴリズムです。

```C++
%%writefile simple_nth_element.h

#include <algorithm>

// 三つの引数をcompの順になるように並べ替える関数
template <class RandomAccessIterator, class Compare>
void sort_3(RandomAccessIterator first, RandomAccessIterator middle, RandomAccessIterator last, Compare comp) {
    if (comp(*first, *middle)) {
        if (!comp(*middle, *last)) {
            std::iter_swap(middle, last);
            if (!comp(*first, *middle)) {
                std::iter_swap(first, middle);
            }
        }
    } else {
        std::iter_swap(first, middle);
        if (!comp(*middle, *last)) {
            std::iter_swap(middle, last);
            if (!comp(*first, *middle)) {
                std::iter_swap(first, middle);
            }
        }
    }
}

// compの順においてn番目までの要素を取得する関数
template <class RandomAccessIterator, class Compare>
void simple_nth_element(RandomAccessIterator first, RandomAccessIterator middle, RandomAccessIterator last, Compare comp) {
    while (3 < last - first) {
        // 先頭、途中、末尾の値を並び替えて、途中の値を使う
        sort_3(first, middle, last - 1, comp);
        auto m = *middle;
        // 値mで前後に分ける
        auto pos = std::partition(first, last, [m, comp](auto v) { return comp(v ,m); });
        // 対象範囲を絞る
        if (pos < middle) {
            if (first == pos) ++first;
            else first = pos;
        } else if (middle < pos) {
            if (last == pos) --last;
            else last = pos;
        }
        else return;
    }
    sort_3(first, middle, last - 1, comp);
}
```

    Overwriting simple_nth_element.h

次が、処理時間を計測するプログラムです。クロック数はいわば手数にあたるものですが、IOの時間が反映されません。これらのアルゴリズムではキャッシュとの兼ね合いなども重要な要素なので、実時間の方に注目すべきです。

```C++
%%writefile time_attack.cpp

#include <algorithm>
#include <vector>
#include <random>

#include "util.h"
#include "simple_partial_sort.h"
#include "simple_nth_element.h"
#include "std_partial_sort.h"

int main(int argc, char **argv) {
    int num = atoi(argv[1]);
    int topk = atoi(argv[2]);
    std::vector<int> a[N], b[N];
    std::vector<int> out(topk, 0);
    std::random_device seed_gen;
    std::mt19937 engine(seed_gen());
    //std::mt19937 engine(0);
    for (int i = 0; i < N; ++i) {
        a[i].resize(num);
        std::generate(a[i].begin(), a[i].end(), engine);
    }

    for (size_t i = 0; i < N; ++i) b[i] = a[i];
    T("std::partial_sort", std::partial_sort(b[i].begin(), b[i].begin() + topk, b[i].end(), std::less<>()));
    P("std::partial_sort", b[0].begin(), 10);

    for (size_t i = 0; i < N; ++i) b[i] = a[i];
    T("std_partial_sort", std_partial_sort(b[i].begin(), b[i].begin() + topk, b[i].end(), std::less<>()));
    P("std_partial_sort", b[0].begin(), 10);

    for (size_t i = 0; i < N; ++i) b[i] = a[i];
    T("simple_partial_sort", simple_partial_sort(b[i].begin(), b[i].begin() + topk, b[i].end(), std::less<>()));
    P("simple_partial_sort", b[0].begin(), 10);

    for (size_t i = 0; i < N; ++i) b[i] = a[i];
    T("std::nth_element", std::nth_element(b[i].begin(), b[i].begin() + topk, b[i].end(), std::less<>()));
    std::sort(b[0].begin(), b[0].begin() + topk);
    P("std::nth_element", b[0].begin(), 10);

    for (size_t i = 0; i < N; ++i) b[i] = a[i];
    T("std_nth_element\t", std_nth_element(b[i].begin(), b[i].begin() + topk, b[i].end(), std::less<>()));
    std::sort(b[0].begin(), b[0].begin() + topk);
    P("std_nth_element\t", b[0].begin(), 10);

    for (size_t i = 0; i < N; ++i) b[i] = a[i];
    T("simple_nth_element", simple_nth_element(b[i].begin(), b[i].begin() + topk, b[i].end(), std::less<>()));
    std::sort(b[0].begin(), b[0].begin() + topk);
    P("simple_nth_element", b[0].begin(), 10);
}
```

    Overwriting time_attack.cpp

```shell
!g++ time_attack.cpp -O3
```

上記の測定プログラムではあえて乱数を固定していません。10回の平均をとるとはいえ、対象の乱数の具合でどれほど影響があるか、見るためです。

それでは1億個の乱数から上位10個を取り出す処理を5回やってみましょう。ちなみに結果を見やすくするため、2回目以降は標準出力を非表示にしています。

```shell
!./a.out 100000000 10
```

    std::partial_sort ave: 0.077587 sec 8191 clock
    std::partial_sort -2147483633 -2147483542 -2147483534 -2147483515 -2147483494 -2147483487 -2147483465 -2147483335 -2147483318 -2147483255
    std_partial_sort ave: 0.065622 sec 13107 clock
    std_partial_sort -2147483633 -2147483542 -2147483534 -2147483515 -2147483494 -2147483487 -2147483465 -2147483335 -2147483318 -2147483255
    simple_partial_sort ave: 0.102721 sec 13107 clock
    simple_partial_sort -2147483633 -2147483542 -2147483534 -2147483515 -2147483494 -2147483487 -2147483465 -2147483335 -2147483318 -2147483255
    std::nth_element ave: 1.356455 sec 137625 clock
    std::nth_element -2147483633 -2147483542 -2147483534 -2147483515 -2147483494 -2147483487 -2147483465 -2147483335 -2147483318 -2147483255
    std_nth_element  ave: 0.072911 sec 13107 clock
    std_nth_element  -2147483633 -2147483542 -2147483534 -2147483515 -2147483494 -2147483487 -2147483465 -2147483335 -2147483318 -2147483255
    simple_nth_element ave: 0.877183 sec 91750 clock
    simple_nth_element -2147483633 -2147483542 -2147483534 -2147483515 -2147483494 -2147483487 -2147483465 -2147483335 -2147483318 -2147483255

```shell
!./a.out 100000000 10 > /dev/null
```

    std::partial_sort ave: 0.075216 sec 13107 clock
    std_partial_sort ave: 0.072400 sec 13107 clock
    simple_partial_sort ave: 0.096547 sec 13107 clock
    std::nth_element ave: 1.000136 sec 104857 clock
    std_nth_element  ave: 0.076756 sec 13107 clock
    simple_nth_element ave: 1.237228 sec 124518 clock

```shell
!./a.out 100000000 10 > /dev/null
```

    std::partial_sort ave: 0.064199 sec 13107 clock
    std_partial_sort ave: 0.073393 sec 13105 clock
    simple_partial_sort ave: 0.091290 sec 9830 clock
    std::nth_element ave: 0.612676 sec 62259 clock
    std_nth_element  ave: 0.077525 sec 8191 clock
    simple_nth_element ave: 0.999661 sec 101580 clock

```shell
!./a.out 100000000 10 > /dev/null
```

    std::partial_sort ave: 0.076138 sec 7782 clock
    std_partial_sort ave: 0.076562 sec 9830 clock
    simple_partial_sort ave: 0.089750 sec 9830 clock
    std::nth_element ave: 0.800072 sec 81919 clock
    std_nth_element  ave: 0.077522 sec 9420 clock
    simple_nth_element ave: 0.771879 sec 78643 clock

```shell
!./a.out 100000000 10 > /dev/null
```

    std::partial_sort ave: 0.070214 sec 13107 clock
    std_partial_sort ave: 0.067098 sec 13107 clock
    simple_partial_sort ave: 0.094695 sec 13107 clock
    std::nth_element ave: 0.635329 sec 63897 clock
    std_nth_element  ave: 0.074179 sec 8191 clock
    simple_nth_element ave: 0.571285 sec 57343 clock

以下のことが分かります。

* 各 partial_sort と std_nth_element() は概ね安定している
* std::nth_element() と simple_nth_element() は対象配列の分布に処理時間が大きく左右される。
* std::partial_sort() と std_partial_sort() はほぼ同等である。
* simple_partial_sort() より std_partial_sort() の方が効率が良い。
* simple_nth_element() より std::nth_element() の方が効率が良い場合が多いが、オーダーはほぼ同等。
* std::nth_element() と simple_nth_element() はTop 10では効率が悪い。

それでは順次、kを大きくして実行してみます。

```shell
!./a.out 100000000 100 > /dev/null
```

    std::partial_sort ave: 0.065807 sec 13107 clock
    std_partial_sort ave: 0.066256 sec 13107 clock
    simple_partial_sort ave: 0.091300 sec 9830 clock
    std::nth_element ave: 0.561723 sec 58982 clock
    std_nth_element  ave: 0.066902 sec 13107 clock
    simple_nth_element ave: 0.600022 sec 65535 clock

```shell
!./a.out 100000000 1000 > /dev/null
```

    std::partial_sort ave: 0.075713 sec 8191 clock
    std_partial_sort ave: 0.076911 sec 8191 clock
    simple_partial_sort ave: 0.099498 sec 13107 clock
    std::nth_element ave: 0.611028 sec 62259 clock
    std_nth_element  ave: 0.072518 sec 13107 clock
    simple_nth_element ave: 1.019500 sec 104857 clock

```shell
!./a.out 100000000 10000 > /dev/null
```

    std::partial_sort ave: 0.086458 sec 9830 clock
    std_partial_sort ave: 0.086798 sec 13107 clock
    simple_partial_sort ave: 0.106478 sec 13107 clock
    std::nth_element ave: 0.601796 sec 60620 clock
    std_nth_element  ave: 0.084962 sec 9830 clock
    simple_nth_element ave: 0.668110 sec 67174 clock

```shell
!./a.out 100000000 100000 > /dev/null
```

    std::partial_sort ave: 0.201937 sec 26214 clock
    std_partial_sort ave: 0.182388 sec 19660 clock
    simple_partial_sort ave: 0.227703 sec 26214 clock
    std::nth_element ave: 0.991851 sec 104857 clock
    std_nth_element  ave: 0.169735 sec 19660 clock
    simple_nth_element ave: 1.047386 sec 209715 clock

```shell
!./a.out 100000000 1000000 > /dev/null
```

    std::partial_sort ave: 1.551150 sec 209715 clock
    std_partial_sort ave: 1.364721 sec 157286 clock
    simple_partial_sort ave: 1.617882 sec 209715 clock
    std::nth_element ave: 0.369516 sec 37683 clock
    std_nth_element  ave: 1.203761 sec 209715 clock
    simple_nth_element ave: 0.893800 sec 91750 clock

```shell
!./a.out 100000000 10000000 > /dev/null
```

    std::partial_sort ave: 13.380789 sec 1677721 clock
    std_partial_sort ave: 12.185713 sec 1677721 clock
    simple_partial_sort ave: 13.654719 sec 1468006 clock
    std::nth_element ave: 1.024504 sec 104857 clock
    std_nth_element  ave: 8.741089 sec 1677721 clock
    simple_nth_element ave: 0.805549 sec 81919 clock

kが100万まではpartial_sort系が圧倒的に速いです。そこを超えてくるとnth_elementの方が相対的に速くなってきます。ただ、top100万超が実用的に必要となる場面は少なそうです。

## 8. ポインターのソート

元の配列は変えずにソートしたい場合、要素へのポインタをソートするにはどうしたらいいでしょうか。独自の比較演算子を定義することで実現できます。見やすくするためエラー処理は省いています。

```C++
%%writefile sort_ptr.cpp
#include <algorithm>
#include <random>
#include <iostream>

template <typename T>
bool greater_ptr(const T* x_ptr, const T* y_ptr) {
    return *x_ptr > *y_ptr;
}

template <typename T>
bool less_ptr(const T* x_ptr, const T* y_ptr) {
    return *x_ptr < *y_ptr;
}

int main(int argc, char **argv) {
    int n = atoi(argv[1]);
    int k = atoi(argv[2]);
    int *a = (int*)malloc(sizeof(int) * n);
    std::mt19937 engine(0);
    std::generate(a, a + n, engine);

    for (int i = 0; i < n; ++i) std::cout << i << "\t" << a[i] << std::endl;
    int **p = (int**)malloc(sizeof(int*) * n);
    for (int i = 0; i < n; ++i) p[i] = a + i;

    std::cout << "greater" << std::endl;
    std::partial_sort(p, p + k, p + n, greater_ptr<int>);
    for (int i = 0; i < k; ++i) std::cout << p[i] - a << "\t" << *p[i] << std::endl;

    std::cout << "less" << std::endl;
    std::partial_sort(p, p + k, p + n, less_ptr<int>);
    for (int i = 0; i < k; ++i) std::cout << p[i] - a << "\t" << *p[i] << std::endl;

    free(a);
    free(p);
}
```

    Overwriting sort_ptr.cpp

```shell
!g++ sort_ptr.cpp ; ./a.out 20 5
```

    0 -1937831252
    1 -1748719057
    2 -1223252363
    3 -668873536
    4 -1706118333
    5 -610118917
    6 -1954711869
    7 -656048793
    8 1819583497
    9 -1616781613
    10 -1520873195
    11 1650906866
    12 1879422756
    13 1277901399
    14 -464831418
    15 243580376
    16 -156067240
    17 1171049868
    18 1646868794
    19 2051556033
    greater
    19 2051556033
    12 1879422756
    8 1819583497
    11 1650906866
    18 1646868794
    less
    6 -1954711869
    0 -1937831252
    1 -1748719057
    4 -1706118333
    9 -1616781613

できました。一点、注意点があります。配列としてvectorも利用できてしまいますが、これはよろしくありません。なぜならvectorの要素へのポインターは変更される可能性があるからです。ご注意ください。

## 9. インデックス番号のソート

ポインターのソートはできましたが、配列要素へのポインターの配列を作らなければなりません。ポインターは8バイト使うので、この配列はかなり大きくなってしまいます。これを節約するためにインデックス番号でソートする方法はないでしょうか。これも、独自比較演算子を使えばできます。元の配列にアクセスする必要があるので、比較演算子のコンストラクタに元の配列を引数で指定します。

インデックス番号のソートなら、vectorでも問題ありません。

```C++
%%writefile sort_index.cpp

#include <algorithm>
#include <random>
#include <iostream>
#include <vector>
#include <numeric>

template <typename T>
struct greater_index {
  const T &vec;
  greater_index(const T& vec) : vec(vec) {}
  bool operator()(const size_t x, const size_t y) {
    return vec[x] > vec[y];
  }
};

template <typename T>
struct less_index {
  const T &vec;
  less_index(const T& vec) : vec(vec) {}
  bool operator()(const size_t x, const size_t y) {
    return vec[x] < vec[y];
  }
};

int main(int argc, char **argv) {
    int n = atoi(argv[1]);
    int k = atoi(argv[2]);
    std::vector<int> a(n);
    std::mt19937 engine(0);
    std::generate(a.begin(), a.end(), engine);

    for (int i = 0; i < n; ++i) std::cout << i << "\t" << a[i] << std::endl;

    std::vector<uint16_t> index(n);
    iota(index.begin(),index.end(), 0);

    std::cout << "greater" << std::endl;
    std::partial_sort(index.begin(), index.begin() + k, index.end(), greater_index<std::vector<int>>(a));
    for (int i = 0; i < k; ++i) std::cout << index[i] << "\t" << a[index[i]] << std::endl;

    std::cout << "less" << std::endl;
    std::partial_sort(index.begin(), index.begin() + k, index.end(), less_index<std::vector<int>>(a));
    for (int i = 0; i < k; ++i) std::cout << index[i] << "\t" << a[index[i]] << std::endl;

}
```

    Overwriting sort_index.cpp

```shell
!g++ sort_index.cpp ; ./a.out 20 5
```

    0 -1937831252
    1 -1748719057
    2 -1223252363
    3 -668873536
    4 -1706118333
    5 -610118917
    6 -1954711869
    7 -656048793
    8 1819583497
    9 -1616781613
    10 -1520873195
    11 1650906866
    12 1879422756
    13 1277901399
    14 -464831418
    15 243580376
    16 -156067240
    17 1171049868
    18 1646868794
    19 2051556033
    greater
    19 2051556033
    12 1879422756
    8 1819583497
    11 1650906866
    18 1646868794
    less
    6 -1954711869
    0 -1937831252
    1 -1748719057
    4 -1706118333
    9 -1616781613

できました。余分に必要な配列は vector\<uint16_t\> なので、ポインターより省スペースです。もちろん、上限があるので、超える場合、適宜uint32_t等に変えてください。

最後に、もっと格好良い書き方を思いついたので記載しておきます。

```C++
%%writefile sort_index.cpp

#include <algorithm>
#include <random>
#include <iostream>
#include <vector>
#include <numeric>

template <typename T>
auto greater_index(const T& vec) {
  return [vec](int left_index, int right_index) {
    return vec[left_index] > vec[right_index];
  };
}

template <typename T>
auto less_index(const T& vec) {
  return [vec](int left_index, int right_index) {
    return vec[left_index] < vec[right_index];
  };
}

int main(int argc, char **argv) {
    int n = atoi(argv[1]);
    int k = atoi(argv[2]);
    std::vector<int> a(n);
    std::mt19937 engine(0);
    std::generate(a.begin(), a.end(), engine);

    for (int i = 0; i < n; ++i) std::cout << i << "\t" << a[i] << std::endl;

    std::vector<uint16_t> index(n);
    iota(index.begin(),index.end(), 0);

    std::cout << "greater" << std::endl;
    std::partial_sort(index.begin(), index.begin() + k, index.end(), greater_index<>(a));
    for (int i = 0; i < k; ++i) std::cout << index[i] << "\t" << a[index[i]] << std::endl;

    std::cout << "less" << std::endl;
    std::partial_sort(index.begin(), index.begin() + k, index.end(), less_index<>(a));
    for (int i = 0; i < k; ++i) std::cout << index[i] << "\t" << a[index[i]] << std::endl;

}
```

    Overwriting sort_index.cpp

```shell
!g++ sort_index.cpp ; ./a.out 20 5
```

    0 -1937831252
    1 -1748719057
    2 -1223252363
    3 -668873536
    4 -1706118333
    5 -610118917
    6 -1954711869
    7 -656048793
    8 1819583497
    9 -1616781613
    10 -1520873195
    11 1650906866
    12 1879422756
    13 1277901399
    14 -464831418
    15 243580376
    16 -156067240
    17 1171049868
    18 1646868794
    19 2051556033
    greater
    19 2051556033
    12 1879422756
    8 1819583497
    11 1650906866
    18 1646868794
    less
    6 -1954711869
    0 -1937831252
    1 -1748719057
    4 -1706118333
    9 -1616781613

それにしてもC++は便利ですね。以上です。
