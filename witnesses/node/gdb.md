---
description: Установка ноды на сервере с ОС Ubuntu 16.04
---

# Нода с отладкой GDB

Бывает, что из-за ошибок демон golosd вылетает с сообщением Segmentation fault или Aborted, в папке /var/lib/golosd появляется core dumped. При этом больше никакой информации. В таком случае пригодится отладка через [gdb](https://ru.wikipedia.org/wiki/GNU_Debugger).

Устанавливаем необходимые пакеты:

```text
sudo apt-get update
```

```text
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

```text
sudo pip3 install gcovr
```

Копируем исходные файлы для сборки ноды из github:

```text
git clone https://github.com/golos-blockchain/golos.git && cd golos
```

```text
git submodule update --init --recursive -f
```

Задаём значения переменных и конфигурируем проект:

```text
mkdir build && cd build
```

```text
cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_GOLOS_TESTNET=FALSE \
    -DBUILD_SHARED_LIBRARIES=FALSE \
    -DLOW_MEMORY_NODE=FALSE \
    -DCHAINBASE_CHECK_LOCKING=FALSE \
    ..
```

Запуск сборки с установкой демона в `/usr/local/`, исполнив:

```text
make -j $(nproc) && sudo make install
```

## Подготовка файлов

```text
mkdir -p ~/golos/build/programs/golosd/witness_node_data_dir/blockchain
```

Копируем в папку `.../golosd/witness_node_data_dir` свой конфиг для ноды, добавив сид-ноды, напр.  
  
`p2p-seed-node = golos1.lexai.host:4243  
p2p-seed-node = golos2.lexai.host:4243`  
  
Копируем в папку `.../golosd/witness_node_data_dir/blockchain` бэкап файлов блоклогс и шаред-мемори, чтобы не терять время на синхронизацию сети \(копии возможно скачать [здесь](https://wiki.golos.id/witnesses/node/guide#ustanavlivaem-nodu)\).

## Запуск GDB

Устанавливаем отладчик gdb

```text
sudo apt-get install gdb
```

Переходим в папку проекта

```text
cd ~/golos/build/programs/golosd
```

Запускаем демон через gdb

```text
gdb ./golosd
```

Включаем сохранение лога \(в файл gdb.txt\) рядом с файлом запуска

```text
set logging on
```

Подтверждаем запуск

```text
run
```

