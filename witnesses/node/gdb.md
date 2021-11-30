---
description: Установка ноды на сервере с ОС Ubuntu 16.04
---

# Нода с отладкой GDB

Бывает, что из-за ошибок демон golosd вылетает с сообщением Segmentation fault или Aborted, в папке /var/lib/golosd появляется core dumped. При этом больше никакой информации. В таком случае пригодится отладка через [gdb](https://ru.wikipedia.org/wiki/GNU\_Debugger).

Устанавливаем необходимые пакеты:

```
sudo apt-get update
```

```
sudo apt-get install -y \
        autoconf \
        automake \
        autotools-dev \
        bsdmainutils \
        build-essential \
        cmake \
        doxygen \
        git \
        ccache \
        libboost-all-dev \
        libreadline-dev \
        libssl-dev \
        libtool \
        ncurses-dev \
        pbzip2 \
        pkg-config \
        python3 \
        python3-dev \
        python3-pip \
        runit
```

```
sudo pip3 install gcovr
```

Копируем исходные файлы для сборки ноды из github:

```
git clone https://github.com/golos-blockchain/golos.git && cd golos
```

```
git submodule update --init --recursive -f
```

Задаём значения переменных и конфигурируем проект:

```
mkdir build && cd build
```

```
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_GOLOS_TESTNET=FALSE \
    -DBUILD_SHARED_LIBRARIES=FALSE \
    -DLOW_MEMORY_NODE=FALSE \
    -DCHAINBASE_CHECK_LOCKING=FALSE \
    ..
```

Запуск сборки с установкой демона в `/usr/local/`, исполнив:

```
make -j $(nproc) && sudo make install
```

## Подготовка файлов

```
mkdir -p ~/golos/build/programs/golosd/witness_node_data_dir/blockchain
```

Копируем в папку `.../golosd/witness_node_data_dir` свой конфиг для ноды, добавив сид-ноды, напр.\
\
`p2p-seed-node = golos1.lexai.host:4243`\
`p2p-seed-node = golos2.lexai.host:4243`\
\
Копируем в папку `.../golosd/witness_node_data_dir/blockchain` бэкап файлов блоклогс и шаред-мемори, чтобы не терять время на синхронизацию сети (копии возможно скачать [здесь](https://wiki.golos.id/witnesses/node/guide#ustanavlivaem-nodu)).

## Запуск GDB

Устанавливаем отладчик gdb

```
sudo apt-get install gdb
```

Переходим в папку проекта

```
cd ~/golos/build/programs/golosd
```

Запускаем демон через gdb

```
gdb ./golosd
```

Включаем сохранение лога (в файл gdb.txt) рядом с файлом запуска

```
set logging on
```

Подтверждаем запуск

```
run
```
