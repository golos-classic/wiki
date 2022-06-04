---
description: Установка ноды на сервере с ОС Ubuntu 18.04
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
git clone https://github.com/golos-blockchain/chain-node.git && cd chain-node
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
mkdir -p ~/chain-node/build/programs/golosd/witness_node_data_dir/blockchain
```

```
sudo cp ~/chain-node/share/golosd/snapshot5392323.json ~/chain-node/build/programs/golosd/ && 
sudo cp ~/chain-node/share/golosd/seednodes ~/chain-node/build/programs/golosd/witness_node_data_dir/ && 
sudo cp ~/chain-node/share/golosd/config/config_witness.ini ~/chain-node/build/programs/golosd/witness_node_data_dir/config.ini
```

Возможно понадобится прописать сид-ноды в конфиг:\
`p2p-seed-node = golos1.lexai.host:4243`\
`p2p-seed-node = golos2.lexai.host:4243`

Копируем в папку `.../golosd/witness_node_data_dir/blockchain` бэкап файлов блоклогс и шаред-мемори, чтобы не терять время на синхронизацию сети (возможно скачать [здесь](https://wiki.golos.id/witnesses/node/guide#ustanavlivaem-nodu)).

## Запуск GDB

Устанавливаем отладчик gdb

```
sudo apt-get install gdb -y
```

Переходим в папку проекта

```
cd ~/chain-node/build/programs/golosd
```

Запускаем демон через gdb

```
gdb ./golosd
```

На вопрос _Quit this debugging session? (y or n), отменяем вводом **n**_

Включаем сохранение лога (в файл gdb.txt) рядом с файлом запуска

```
set logging on
```

Подтверждаем запуск

```
run
```
