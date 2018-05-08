Title: mpirun と wrapper
Date: 2016-05-08 10:20
Category: MPI
Tags: mpi
Authors: Hidemoto Nakada

# mpirunとwrapper

mpirunでジョブを起動する際、通常は
```
mpirun MPI_BINARY
```
のように引数にmpiccでコンパイルしたバイナリを指定するわけだが、
ここに非MPIプログラムを書いて、そこからMPIプロセスを起動しても
大丈夫なのかを確認して見た。

## mpiプログラム

テストプログラムはこんな感じ。
```
#include <mpi.h>
#include <stdio.h>

int main(int argc, char ** argv){
    int rank = -1;
    MPI_Init(& argc, & argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    printf("rank = %d\n", rank);
    MPI_Finalize();
}
```

## wrapper
wrapper はpython。上のプログラムを起動するだけ。
```
import subprocess

def main():
    p = subprocess.Popen("./mpitest")
    p.wait()

main()
```

## テスト
pythonのwrapperをmpirunで起動する。
```
$ mpirun -np 2 python launchtest.py 
rank = 0
rank = 1
```

うむ動いた。

## どうなってるのか？
よくわからないが、おそらくmpirunが環境変数にいろいろ値をセットしていて、
それを見てmpiライブラリが何かしているのだろう。

下のようにして環境変数を見てみると、山のようにいろいろでてくる。
```
$ mpirun --host localhost -np 1 printenv | grep OMPI
OMPI_COMMAND=printenv
OMPI_MCA_orte_precondition_transports=e86c6dc6f222e793-e5e9486159da2671
OMPI_MCA_orte_peer_modex_id=0
OMPI_MCA_orte_peer_init_barrier_id=1
OMPI_MCA_orte_peer_fini_barrier_id=2
....
```

問題はこの方法にどのくらいポータビリティがあるのか、だなあ。。