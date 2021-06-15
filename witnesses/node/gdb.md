---
description: Установка ноды на сервере с ОС Ubuntu 16.04
---

# Нода с GDB

Бывает, что из-за ошибок демон golosd вылетает с сообщением Segmentation fault \(core dumped\) или Aborted \(core dumped\). При этом больше никакой информации может не быть. В таком случае пригодится отладка через [gdb](https://ru.wikipedia.org/wiki/GNU_Debugger).

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

## Запуск GDB

Устанавливаем отладчик gdb:

```text
sudo apt-get install gdb
```

Переходим в папку проекта

```text
cd programs/golosd
```

Создаём папку

```text
mkdir witness_node_data_dir
```

Копируем в неё \(путь `golos/build/programs/golosd/witness_node_data_dir` \) свой конфиг для ноды, а создав вложенную папку `blockchain` закидываем туда бэкап блоклогс и шаред-мемори файлов.  
  
Запускаем демон через gdb: 

```text
gdb ./golosd
```

Подтверждаем запуск

```text
run
```

