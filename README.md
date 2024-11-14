<h2>
    Tareo's Learning Notes
</h2>

這邊是我做筆記的地方，大部分文章應該都要有登入 HackMD 才能看到

OpenBMC : https://github.com/openbmc

AspeedTech-BMC : https://github.com/AspeedTech-BMC

IPMI specification : [Sepcification Link](https://www.intel.com.tw/content/www/tw/zh/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)

<h2>
    Linux 基礎知識
</h2>

[Linux basic knowledge 這篇超好用](<https://hackmd.io/@combo-tw/Linux-%E8%AE%80%E6%9B%B8%E6%9C%83/%2F%40combo-tw%2FHyJXuuy8H>)

![image](https://hackmd.io/_uploads/S1cB4AhiA.png)

<h2>
    Intel Platforms
</h2>

代號表

![image](https://hackmd.io/_uploads/BkiivKuoA.png)

注意: 帶有訊號 `*` 表示為計劃中的類型

Intel Chipset 是專門為Intel 處理器設計的，用來連接CPU 與其他裝置如內存，顯示卡等（​​原南橋晶片，現在的PCH 晶片

通常應該可以透過 Schematic 上的 Chipset 來判斷他是哪個 Platforms

BHS BeechnutCity 
BHS ArcherCity
EGS ArcherCity

:::info
**EGS** - Eagle Stream
**BHS** - Birch Stream
:::

<h2>
    AMD Server code
</h2>

![image](https://hackmd.io/_uploads/rk3-Hzmf1x.png)


<h2>
    C++ Compile process
</h2>

https://medium.com/@alan81920/c-c-%E4%B8%AD%E7%9A%84-static-extern-%E7%9A%84%E8%AE%8A%E6%95%B8-9b42d000688f

<h2>
    Dediprog
</h2>

燒錄 EEPROM 用, BMC or BIOS 都可以

<h2>
    readlf
</h2>

有時候需要瞭解某個執行檔或共享函式庫執行時所依賴的函式庫是哪些，就可以用 readelf 或者 ldd 來查看

```shell
readelf -d <executable binary|library> | grep NEEDED
```

例如查看 gdb 執行檔的所依賴的函式庫是哪些，這些 Shared library 顯示 NEEDED 表示執行時需要

```shell
$ readelf -d `which gdb` # 或者 readelf -d /usr/bin/gdb

Dynamic section at offset 0x629d68 contains 36 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libreadline.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libz.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libdl.so.2]
 0x0000000000000001 (NEEDED)             Shared library: [libncurses.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libtinfo.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libpython3.5m.so.1.0]
 0x0000000000000001 (NEEDED)             Shared library: [libpthread.so.0]
 0x0000000000000001 (NEEDED)             Shared library: [libexpat.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [liblzma.so.5]
 0x0000000000000001 (NEEDED)             Shared library: [libbabeltrace.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libbabeltrace-ctf.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0x45a8d8
 0x000000000000000d (FINI)               0x79ea6c
#...略
```

例如查看 `libpthread.so` 函式庫的所依賴的函式庫是哪些，這些 Shared library 顯示 NEEDED 表示執行時需要

```shell=
$ readelf -d `gcc -print-file-name=libpthread.so.0`
# 或者 readelf -d /lib/x86_64-linux-gnu/libpthread.so.0

Dynamic section at offset 0x17d50 contains 31 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [ld-linux-x86-64.so.2]
 0x000000000000000e (SONAME)             Library soname: [libpthread.so.0]
 0x000000000000000c (INIT)               0x5580
 0x000000000000000d (FINI)               0x12ad4
#...略
```

<h2>
    7-bit and 8-bit Address
</h2>

**8-bit** Address 比 **7-bit** Address 多了一位代表 **R/W** 位

**Example (0x2C) :** 

`0x2C` 是 7-bit 地址。
`0x58` 是 8-bit 地址，其中包含了讀/寫位。

>**7-bit 地址：**
>I²C 的設備地址本質上是 7 位長。
>例如，`0x2C` 是一個 7-bit 地址（以二進位表示為 00101100）。

>**8-bit 地址：**
>I²C 通訊傳輸時，會將這個 7-bit 地址向左移動 1 位（左移一位後右邊有空出的一位）。
>例如，0x2C 變成了 01011000。
>
>這個移動後的第 8 位（最右側）會被用來表示 讀/寫位：
>* 0 表示寫操作（Write）。
>* 1 表示讀操作（Read）。
>
>**以 0x2C 這個 7-bit 地址為例：**
>
>* 想對該地址進行 **Write** 操作時，R/W位為 0，形成 8-bit 地址 **01011000**，即 **0x58**。
>* 想對該地址進行 **Read** 操作時，R/W位為 1，形成 8-bit 地址 **01011001**，即 **0x59**。

<h2>
    Ubuntu add command
</h2>

在 `/usr/local/bin` 底下新增

![image](https://hackmd.io/_uploads/BkEWlqXeJg.png)

或是

```
nano ~/.zshrc

* method 1 : Alias（將 mycommand 映射為某個具體命令）：
alias mycommand='echo "Hello, World!"'

* method 2 : or Add function
mycommand() {
  echo "Hello, World!"
}

* then 
source ~/.bashrc  # 或 ~/.zshrc
```

<br>

![image](https://hackmd.io/_uploads/S1Nzxecaa.png)
