Title: RustのClosureとCopy/Clone
Date: 2016-05-06 10:20
Category: Rust
Tags: rust
Authors: Hidemoto Nakada

# RustのClosureとCopy

RustのClosureはCopyを実装できないらしい。
一方で普通の関数では、問題なく自動的に実装されるようだ。

```
fn spawner_multi<F> (handler : F ) -> () 
where F : Fn() -> () + Send + Sync + Copy + 'static 
{
	 thread::spawn( handler );
	 thread::spawn( handler );
	 thread::spawn( handler );
}
```
こういう、受け取ったFnを復数回spawnする関数を用意しておいて、

```
fn handler() -> () {
	 println!{"in the handler"};
}

fn main() {
	 spawner_multi( handler );
}
```
とするのはOKなのだが、

```
fn main() {
	 spawner_multi( || handler() );
}
```
とすると、

```
77 |      spawner_multi( || handler() );
   |      ^^^^^^^^^^^^^ the trait `std::marker::Copy` is not implemented for `[closure@src/main.rs:77:18: 77:30]`
```

のようなエラーがでてコンパイルできない。

関数はCopyなのだが、関数を呼び出すだけのClosureでもCopyにはならないということらしい。
言われてみれば、関数は単なるポインタっぽいのでビットパターンコピーでCopyできるけど、
Closureはもう少し複雑な何かなので、Copyできないのも不思議ではないのかも。

## Cloneだと？
では、Cloneにしてみる。明示的に`clone`を呼び出す。

```
fn spawner_multi<F> (handler : F ) -> () 
where F : Fn() -> () + Send + Sync + Clone + 'static 
{
	 thread::spawn( handler.clone() );
	 thread::spawn( handler.clone() );
	 thread::spawn( handler.clone() );
}
```
これでも、関数はOKだがClosureだと動かない。
Closureは自動的にCloneにならないということなのだろうが、
明示的にCloneにする方法もわからない。

一般論として、キャプチャされた変数がすべてCloneなのであれば、
ClosureもCloneになってくれてもいいような気がする。


## Issueがあった

ぐぐってみたところ、Issueが上がっていた。
[Make Clone a lang-item and implement it for closures whose members are all Clone #1369](https://github.com/rust-lang/rfcs/issues/1369)、
[Tracking issue for RFC 2132: copy_closures/clone_closures #44490](https://github.com/rust-lang/rust/issues/44490)。

2018年3月25日にCloseされている。この日は1.25.0がでた日だ。
1.25.0では動かないし、release noteにも載っていない。
次の1.26では使えるようになるのだろうか。

