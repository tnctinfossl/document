# Rustのすゝめ
## 序文
　Linuxのカーネルプログラムには、一般にC言語が使われています。しかし、C言語及びC++はmallocしてfreeをし忘れるメモリーリーク(*)やマルチスレッドプログラミングにおいて、書き込み中のデータを読み込んでしまい整合性(*2)が取れなくなるといった問題を防ぐことが困難であり大規模なプログラムに向かないという欠点があります。
　そこで、近年OSの実装(*3)にも使われているRustを採用することで言語レベルでこれらの問題(*4)を解決した。そして、関数や変数に対するアトリビュート機能や関数型プログラミングにより記述を簡素化することができる。更に、バックエンドがLLVM(*5)で有ることがから、C,C++と同等の速度かつ多様なプラットホームで動作できる。

*　c++ではc++11からサポートされたスマートポインタが存在するが、記述が冗長になりがちであり、更に習得が困難である。
*2 mutexを使い整合性を取る方法もサポートされているが、そのmutexを使わずにアクセスできてしまう。
*3 https://www.redox-os.org/jp/
*4 ただし非管理領域ではそれらの違反を行える。
*5 LLVMはclangやrustに使われているコンパイラのバックエンド

## 学習サイト
基本的にこれを読めばOK
https://doc.rust-jp.rs/book/second-edition/

学んでほしいこと

+ C及びC++との違い:関数と制御構文、構造体、列挙型、メソッド、インターフェイスの書き方
+ メモリ管理と同期: Box, Rc, channel, mutex, Arc
+ 関数型プログラミング: エラーハンドリング(Some<T>, Result<R,L>), データの変換(map, reduce)
+ その他: jsonの読み書き, イテレータ,crateとエントリーポイント(main.rs, lib.rs)

## 確認テスト(付録)
テスト

  
実行結果がイメージできれば十分かな
```rust
fn main(){
  let sum = (1..100).fillter(|x|*x%2==0).map(|x|x*x).sum();//特に深い意味はない
  println!("{}",sum)
}
```
```rust
fn main (){
  let fizz_buzz =(1..100).map(|x|match x%15{
    0=>"Fizz Buzz".to_owned(),
    5|10=>"Buzz".to_owned(),
    3|6|9|12~>"Fizz".to_owned(),
    _=>format!("{}",x)
  }.reduce(String::new()|x,y|format!("{},{}",x,y));
  println!("{}",fizz_buzz);
)
```
```rust
fn trouble()->Some<i32>{
  None
}
 
fn main (){
  if let Some(x)= trouble(){
    println!("{}",x);
  }else{
    println!("none");
  }
}
```
```rust
use std::sync::mpsc;
use std::thread;
 
fn main (){
  let (tx,rx) = mpsc::channel();
  thread::spawn(move||{
    tx.send(10).unwrap();
  });  
 
  if let Ok(v)=rx.recv(){
    println!("{}",v);
  }else{
    println!("error");
  }
}
```
```rust
trait Animal{
  fn say();
}
 
struct Dog{}
struct Cat{}
 
impl Animal for Dog{
  fn say(){println!("wan");}
}
 
impl Animal for Cat{
  fn say(){println!("nya");}
}
 
fn <A:Animal>static_action(animal:&A){animal.say();}
fn dynamic_action(animal:&Box<dyn Animal>){animal.say();}
 
fn main (){
  static_action(Dog{});
  dynamic_action(&Box::new(Cat{}));
}
```
```rust
//コンパイルできるか?
fn main (){
  let a=Box::new(10);
  let b=a;
  let c=a;
}
```
```rust
//コンパイルできるか?
fn main (){
  let a="hello".to_str();
  let b= "world".to_str();
  let c=a+&b;
  let d=a;
}
```