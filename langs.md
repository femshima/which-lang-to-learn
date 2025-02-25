メモ：
## 発表に入れるやつ
- Python
- Julia
- C++
- Rust
- JS
- Haskell

## 実行環境について
サンプルコードの実行時間は以下の環境で計測しました
- CPU: Ryzen 7 PRO 4750G
- メモリ: 32GB
- OS: Manjaro Linux

## スクリプトみの強いやつ
### Python
  - とてもおそい(最速組比20倍以上)
  - PyPy(高速な実装)とかCython(一部コードをCに変換するツール)を使えば人道的な実行時間で済む
  - やれと言わなくてもやることになる
  - ライブラリが豊富
  - 比較的簡単
  - 型アノテーションはあるらしい
  - 京大の資料読め

code:
```py
import math

print("start")
MAX = 100000000

sieve = [True]*(MAX+1)

sieve[0] = False
sieve[1] = False

for i in range(2,int(math.sqrt(MAX))):
    if sieve[i]:
        for j in range(i*i,MAX+1,i):
            sieve[j] = False

primes = [0]*(MAX+1)
pcount = 0

for i in range(2,MAX+1):
    if sieve[i]:
        primes[pcount] = i
        pcount += 1

print(primes[pcount-1])
print("end")
```

result:
```
$ :
$ time python main.py
start
99999989
end

real	0m27.172s
user	0m26.515s
sys	0m0.509s
```

同じコードでもPyPyで実行するとだいぶマシになる

result:
```
$ :
$ time pypy main.py
start
99999989
end

real	0m3.982s
user	0m3.308s
sys	0m0.637s
```

Cythonで重い部分をCに変換しても速くなる

code:
```py
cimport numpy as np
import numpy as np
import math
import cython

def main():
    cdef:
        int MAX, TRUE, FALSE, pcount
        int[:] sieve
        int[:] primes
        int i, j

    MAX = 100000000
    TRUE  = 1
    FALSE = 0

    sieve = np.full(MAX+1, TRUE, dtype="int32") 
    
    sieve[0] = FALSE
    sieve[1] = FALSE
    
    for i in range(2,int(math.sqrt(MAX))):
        if sieve[i]:
            for j in range(i*i,MAX+1,i):
                sieve[j] = FALSE
    
    primes = np.zeros(MAX+1, dtype="int32") 
    pcount = 0
    
    for i in range(2,MAX+1):
        if sieve[i]:
            primes[pcount] = i
            pcount += 1
    
    return primes, pcount
```

result:
```
$ python setup.py build_ext --inplace
$ time python cymain.py
start
99999989
end

real	0m3.788s
user	0m3.706s
sys	0m1.112s
```

### PHP
  - おそい
  - やれとは全く言わないけどたのしいよ
  - Web屋さん(サーバーサイド)専用
{sample:php}

## コンパイルして使うやつ
### C
  - 最速組
  - 工学erはやることになりそう
  - 静的型、型推論なし
  - 自己責任系プログラミング言語

code:
```c
#include <stdio.h>
#include <stdbool.h>

const int MAX = 100000000;

int main(){
    static bool sieve[MAX+1];
    static int primes[MAX+1];
    puts("start");
    for(int i=0; i<=MAX; i++) sieve[i]=true;
    sieve[0] = false;
    sieve[1] = false;
    for(int i=0; i<=MAX; i++){
        if(sieve[i]){
            for(int j = i*i; j<=MAX; j+=i) sieve[j] = false;
        }
    }
    int pcount=0;
    for(int i=0; i<=MAX; i++){
        if(sieve[i]){
            primes[pcount]=i;
            pcount++;
        }
    }

    printf("%d\n", primes[pcount-1]);
    puts("end");
    return 0;
}
```

result:
```
$ clang   main.c   -O3
$ time ./a.out
start
99999989
end

real	0m0.854s
user	0m0.807s
sys	0m0.029s
```


### C++
  - 最速組
  - 競プロerはやっとけ
  - 自分のこと現代的だと思ってる古参言語
  - 書き味はいたって素直。自分の足を撃ち抜く権利はあなたの手に託されています
  - 静的型、型推論ややあり
  - Cにない型がいっぱいある

コンパイル時に最適化フラグを渡さないと10倍以上遅い(一敗)

code:
```cpp
#include <vector>
#include <iostream>

using namespace std;

const int MAX = 100000000;

int main(){
    cout << "start" << endl;
    vector<bool> sieve(MAX+1, true);
    sieve[0] = false;
    sieve[1] = false;
    for(auto i=0; i<=MAX; i++){
        if(sieve[i]){
            for(auto j = i*i; j<=MAX; j+=i) sieve[j] = false;
        }
    }
    vector<int> primes(MAX+1);
    int pcount = 0;
    for(auto i=0; i<=MAX; i++){
        if(sieve[i]){
            primes[pcount] = i;
            pcount++;
        }
    }

    cout << primes[pcount-1] << endl;
    
    cout << "start" << endl;
    return 0;
}
```

result:
```
$ clang++ main.cpp -O3
$ time ./a.out
start
99999989
start

real	0m0.774s
user	0m0.662s
sys	0m0.109s
```

  
### Rust
  - 最速組
  - やれ
  - 2言語目推奨
  - プログラマのことを信頼していない
  - 型が強い
  - 静的型、型推論強い
  - 新世代の勝者？
  - パッケージ管理ツールが優秀
  - ライブラリが豊富
  - ビルドが遅い（特に初回）
  - The Rust Programming Language 読んどけ

code:
```rs
const MAX :usize = 100000000;
fn main() {
    println!("start");

    let mut sieve = vec![true; MAX+1];
    sieve[0] = false;
    sieve[1] = false;
    for i in 2..=(f32::sqrt(MAX as f32) as usize) {
        if sieve[i]{
            for j in (i*i..=MAX).step_by(i) {
                sieve[j] = false;
            }
        }
    }

    let mut primes = vec![0; MAX+1];
    let mut pcount = 0;
    for (i,s) in sieve.iter().enumerate() {
        if *s {
            primes[pcount] = i;
            pcount += 1;
        }
    }
    println!("{}",primes[pcount-1]);
    println!("end");
}
```

result:
```
$ cargo build --release
$ time ./target/release/rs
start
99999989
end

real	0m0.901s
user	0m0.837s
sys	0m0.046s
```


## JIT(ジャストインタイムコンパイル)を使うやつ
### Julia
  - 事前にコンパイルする必要がない
  - が、JITのおかげで比較的速い（最速組には勝てない）
  - PyCallでpythonのライブラリが使える
  - 文法はシンプル
  - 数式・行列が扱いやすい
  - numpyやmatplotlibがpythonよりだいぶ使いやすい
  - 1-indexed、縦行列基本など癖がある（数式をそのまま使えるから数学屋とかシミュレーション屋には便利）
  - Jupyter notebookで使える
  - ちゃんと考えてコーディングしないといまいち速度が出ない
  - クラスはない（Pythonのサンプルコードを脳死で移植はできない）（構造体で似たようなものは作れる）
  - PythonとCとMATLABの良いところどり的存在 by あなばす
  - いまのところ入門向けの良書はあまりない
  - 公式wikiはわかりやすい。

コード全体を関数で包まないと死ぬほど遅くなるので注意（一敗）

code:
```jl
function main()
    MAX = 100000000
    
    println("start")
    
    sieve = fill(true,MAX)
    
    sieve[1] = false
    for i :: Int64 in 2:Int64(floor(sqrt(MAX)))
        if sieve[i]
            for j in (i*i):i:MAX
                sieve[j] = false
            end
        end
    end
    
    
    primes = fill(0,MAX)
    pcount = 1
    for i in 2:MAX
        if sieve[i]
            primes[pcount] = i
            pcount += 1
        end
    end
    
    println(primes[pcount-1])
    
    println("end")
end
main()
```

result:
```
$ julia main.jl
$ time julia main.jl
start
99999989
end

real	0m1.266s
user	0m0.965s
sys	0m0.296s
```


### Java
  - 五十歩百歩組
  - 業務で使われがち
  - ザ・クラスベースオブジェクト指向
  - 静的型
{sample:java}
### C#
  - 五十歩百歩組2
  - MS製のJava
  - Win向けのGUIアプリを書くのが簡単（後述のVB.netとF#も）
  - Unityでつかうらしい
  - 静的型、型推論ややあり
{sample:cs}
### VB.net
  - C#。
  - C#(Basic風味)。
{sample:vb}

## 関数型プログラミングに対するサポートが強いやつ
### OCaml
  - 五十歩百歩組3
  - ML族
  - 非純粋
  - 静的型、型推論強い
{sample:ocaml}
### Haskell
  - 五十歩百歩組4
  - †純粋†関数型言語
  - 中二病患者におすすめ
  - 副作用の詳細な制御、
  - こみいった副作用に対する制御とか複雑なルールの組み合わせとかに滅法強い
  - ビルドが遅い（特に初回）
  - 静的型、型推論強い

最適化すると十分速いが、知識と試行錯誤が必要

code:
```hs
module Main where

import Lib
import qualified Data.Vector.Unboxed as V
import qualified Data.Vector.Unboxed.Mutable as MV
import Data.Foldable
import Control.Monad
import Control.Monad.ST
import Data.STRef
import GHC.Float

num :: Int
num = 100000000

main :: IO ()
main = do
  putStrLn "start"
  let ps = generatePrimes num
  print $ V.last ps
  putStrLn "end"


generatePrimes :: Int -> V.Vector Int
generatePrimes max =
  let sieve = runST (do
          msieve <- MV.replicate (max+1) True
          MV.write msieve 0 False
          MV.write msieve 1 False
          
          forM_ [2..(double2Int $ sqrt $ fromIntegral max)] $ \i -> do
            isprime <- MV.unsafeRead msieve i
            when isprime (forM_ [i..max `div` i] (\j -> MV.unsafeWrite msieve (i*j) False))
          
          V.unsafeFreeze msieve
        )

      primes = runST (do
          mprimes <- MV.replicate (max+1) 0

          pcount <- foldM (\pcount i -> do
            let isprime = V.unsafeIndex sieve i
            if isprime
            then (do
              MV.unsafeWrite mprimes pcount i
              pure (pcount+1)
              )
            else pure pcount
            ) 0 [2..max]
          V.unsafeFreeze $ MV.unsafeSlice 0 pcount mprimes
        )
  --in V.filter (V.unsafeIndex sieve) (V.generate (max+1) id)
  in primes
```

result:
```
$ stack install --local-bin-path ./
$ time ./hs-exe
start
99999989
end

real	0m1.036s
user	0m0.747s
sys	0m0.268s
```

### Lisp
  - 古代の関数型言語
  - アーティファクト
  - 方言が多い
  - きもいモンスター
  - かっこかっこかっこかっこ

## ブラウザでつかえるやつ
### javascript
  - 五十歩百歩組6（なんで？？？きみ動的型のスクリプト言語だろ？？？？？）
  - GUI作りたくなったらHTML+CSS+JSでやるのが一番かんたんかもしれない
  - サーバーサイドもスタンドアロンも書ける
  - 動的型、型アノテーションすらない
  - 愛すべきカス

TypedArray使ったら最速組と張り合える速さが出た
普通の配列を使ったらメモリ確保ができなくなって落ちた

code:
```js
console.log("start");

const max = 100000000;

let sieve = new Int8Array(max+1);
for(let i=2; i<=max; i++) sieve[i] = 1;

for(let i=2; i<=(Math.floor(Math.sqrt(max)) | 0); i++){
    if(sieve[i]){
        for(let j=i*i; j<=max; j+=i) sieve[j] = false;
    }
}

let primes = new Int32Array(max+1);
let pcount = 0;

for(let i=2; i<=max; i++){
    if(sieve[i]){
        primes[pcount] = i;
        pcount++;
    }
}

console.log(primes[pcount-1]);

console.log("end")
```

result:
```
$ :
$ time node main.js
start
99999989
end

real	0m0.915s
user	0m0.866s
sys	0m0.050s
```

### Typescript
  - AltJSのデファクトスタンダード、型のあるJS
  - 型がある！！！しかも強い！！！！！！！（とても重要）
{sample:ts}
### (WebAsssamplery)
  - JS一族ではない
  - 他の言語(RustとかC++とか)からWASMにコンパイルしてブラウザで使える
### coffeescript
  - Rubyのようななにか
{sample:coffee}
### purescript
  - AltJSの異端児、Haskellの生き写し
{sample:purs}
### scala.js
  - scalaがjsにコンパイルされる
{result:scjs}
### GHCjs
  - Haskellがjsにコンパイルされる
{sample:hsjs}
### js_of_ocaml
  - OCamlがjsにコンパイルされる
{sample:jsocaml}
{sample:ocjs}

## 統計とかシミュレーションに使うやつ
### R
  - 統計にめっちゃ強い
  - 検定がコマンド一発でできる
  - 図が綺麗で細かく指定できる
  - データフレームが使いやすい
  - 機械学習のライブラリが多い
  - 複雑なことをすると遅い
  - 検索がしにくい

code:
```r
main <- function(){
    print("start")
    
    max <- 100000000
    
    sieve <- rep(TRUE, max)
    
    sieve[1] = FALSE
    
    for(i in 1:floor(sqrt(max))){
        if(sieve[i]){
            for(j in seq(i*i,max,by=i)) sieve[j] = FALSE
        }
    }
    
    primes <- rep(0, max)
    pcount <- 0;
    
    for(i in 1:max){
        if(sieve[i]){
            primes[pcount] = i
            pcount = pcount + 1
        }
    }
    
    print(primes[pcount-1])
    
    print("end")
}

main()
```

result:
```
$ :
$ time Rscript main.r
[1] "start"
[1] 99999989
[1] "end"

real	0m15.485s
user	0m14.151s
sys	0m1.244s
```

### MATLAB
  - 数式とかシミュレーションが強い
  - グラフィクスが強い
  - 有料だけど東大生は使える
  - そんなに速くはない
### Fortran
  - すごく速いらしい
  - スパコンでよく使われている
  - 文法が独特

なにを間違えたのか大して速くならなかった

code:
```f95
program eratosthenes
    implicit none
    
    integer,parameter :: pmax = 100000000
    integer :: i
    integer :: j
    integer :: pcount = 1
    logical,allocatable,dimension(:) :: sieve
    integer,allocatable,dimension(:) :: primes
    allocate(sieve(pmax))
    sieve=.true.
    allocate(primes(pmax))
    primes=0
   
    print *, "start"
    do i=2 , int(sqrt(real(pmax)))
        if(sieve(i))then
            do j = i*i , pmax , i
                sieve(j) = .false.
            end do
        end if
    end do

    do i=2 , pmax
        if(sieve(i))then
            primes(pcount) = i
            pcount = pcount + 1
        end if
    end do

    print *, primes(pcount-1)

    print *, "end"
end program eratosthenes
```

result:
```
$ gfortran -Ofast main.f95
$ time ./a.out
 start
    99999989
 end

real	0m1.280s
user	0m1.158s
sys	0m0.116s
```

 
## 速度ランキング
実行時間：
| rank | lang | time |
| - | - | - |
| 1 | C++ | 0.774 sec. |
| 2 | C | 0.854 sec. |
| 3 | Rust | 0.901 sec. |
| 4 | JS | 0.915 sec. |
| 5 | Haskell | 1.036 sec. |
| 6 | Julia | 1.266 sec. |
| 7 | Fortran | 1.28 sec. |
| 8 | Cython | 3.788 sec. |
| 9 | PyPy | 3.982 sec. |
| 10 | R | 15.485 sec. |
| 11 | Python | 27.172 sec. |

CPU時間：
| rank | lang | time |
| - | - | - |
| 1 | C++ | 0.662 sec. |
| 2 | Haskell | 0.747 sec. |
| 3 | C | 0.807 sec. |
| 4 | Rust | 0.837 sec. |
| 5 | JS | 0.866 sec. |
| 6 | Julia | 0.965 sec. |
| 7 | Fortran | 1.158 sec. |
| 8 | PyPy | 3.308 sec. |
| 9 | Cython | 3.706 sec. |
| 10 | R | 14.151 sec. |
| 11 | Python | 26.515 sec. |


## 貢献者一覧
- 筆者 
  - 説明: C, C++, Rust, Python, Haskell, Fortran, JS, PHP
  - サンプル: C, C++, Rust, Python, Haskell, Fortran, JS, R
  - 一言: Haskellはいい言語ですよ、やれ！お前も蓮沼に落ちろ！！！
- あなばす
  - 説明: Julia, Lisp, R, MATLAB, Fortran
  - 一言: Julia最高！Juliaしか勝たん！
- 綿谷 雫
  - サンプル: Fortran
  - 一言: 古典を学ぶことは物事の根幹に触れることであり，そこから派生してできたものの理解が深まります．古典語をやりましょう．