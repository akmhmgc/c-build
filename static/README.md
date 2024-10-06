```
docker compose up pusher
pusher-1  | # command-line-arguments
pusher-1  | /usr/local/go/pkg/tool/linux_arm64/link: running gcc failed: exit status 1
pusher-1  | /usr/bin/ld: /go/pkg/mod/github.com/confluentinc/confluent-kafka-go@v1.5.2/kafka/librdkafka/librdkafka_glibc_linux.a(rdkafka_error.o): Relocations in generic ELF (EM: 62)
pusher-1  | /usr/bin/ld: /go/pkg/mod/github.com/confluentinc/confluent-kafka-go@v1.5.2/kafka/librdkafka/librdkafka_glibc_linux.a(rdkafka_error.o): Relocations in generic ELF (EM: 62)
pusher-1  | /usr/bin/ld: /go/pkg/mod/github.com/confluentinc/confluent-kafka-go@v1.5.2/kafka/librdkafka/librdkafka_glibc_linux.a: error adding symbols: file in wrong format
pusher-1  | collect2: error: ld returned 1 exit status
```

上記を再現するために、arm64環境で静的ライブラリを作成した上でローカル（amd64環境）でリンクしてビルドした上で呼び出してみる。

```
docker run -it --name cross-compile-x86 debian bash
apt update
apt install gcc-x86-64-linux-gnu build-essential

docker cp ./mylib.c cross-compile-x86:/root/
docker exec -it cross-compile-x86 bash
x86_64-linux-gnu-gcc -c /root/mylib.c -o /root/mylib.o
ar rcs /root/libmylib.a /root/mylib.o

docker cp cross-compile-x86:/root/libmylib.a ./
```

上記で作成した静的ライブラリを利用してホスト環境(arm64)main.cをリンクしようとするエラーがでる。

```
gcc -o main main.c -L. -lmylib
```

