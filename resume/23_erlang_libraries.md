## Erlang libraries

Elixir menyedaiakan interoperability dengan Erlang libraries. Faktanya, Elixir tidak menyarankan membungkus Erlang libararies dan berinteraksi secara langsung dengan Erlang code. Di sesi ini, kita akan mempersentasikan beberapa functionality Erlang yang umum dan berguna yang tidak ditemukan di Elixir.

Erlang module mempunya konvensi penamaan yang berbeda dengan Elixir dan dimulai dengan lowercase. Di kedua kasus, nama modul adalah atoms dan kita memanggil function dengan mengirimkan ke nama modul:

```elixir
is_atom(String)

true

String.first("hello")
"h"

is_atom(:binary)
true

:binary.first("hello")
104
```

Nantinya ketika semakin mahir Elixir, bisa kita explore lebih dalam tentang Erlang di [STDLIB reference Manual](http://www.erlang.org/doc/apps/stdlib/index.html)

**The binary module**

Built-in Elixir string module menghandle binaries yang diencode UTF-8. module `:binary` sangat berguna ketika berurusan dengan binary data yang tidak perlu diencode UTF-8

```elixir
String.to_charlist("Ø")
[216]
:binary.bin_to_list("Ø")
[195, 152]
```

**Formatted text output**

Elixir tidak berisi function yang mirip dengan `printf` yang ditemukan di C dan bahasa pemrograman lainnya. Untungnya, Erlang standard library functions `:io.format/2` dan `:io_lib.format/2` mungkin bisa digunakan. Format yang pertama ke output terminal, sedangkan format yang kedua ke sebuah iolist. Format specifiers berbeda dengan `printf`, [Detailnya bisa dibaca di sini](https://www.erlang.org/doc/man/io.html#format-2)

```elixir
:io.format("Pi is approximately given by:~10.3f~n", [:math.pi])
Pi is approximately given by:     3.142
:ok
to_string(:io_lib.format("Pi is approximately given by:~10.3f~n", [:math.pi]))
"Pi is approximately given by:     3.142\n"
```

**The crypto module**

`:crypto` module berisi function hashing, digital signature, encryption dan lainnya:

```elixir
Base.encode16(:crypto.hash(:sha256, "Elixir"))
"3315715A7A3AD57428298676C5AE465DADA38D951BDFAC9348A8A31E9C7401CB"
```

module `:crypto` adalah bagian dari aplikasi `:crypto` yang dikirim bersama dengan Erlang. Ini berarti kita harus me-list aplikasi `:crypto` sebagai sebuah aplikasi tambahan di konfigurasi project. Untuk melakukan ini, edit `mix.exs`:

```elixir
def application do
  [extra_applications: [:crypto]]
end
```

module apapun yang tidak termasuk bagian dari `:kernel` atau `:stdlib` aplikasi Erlang harus secara explisit dilist di `mix.exs`. Kamu dapat menemukan nama aplikasi dari module Erlang apapun di Erlang dokumentasi, secara langsung di bawah logo Erlang di sidebar.

**The disgraph module**

`:disgraph` dan `:disgraph_utils` module berisi function untuk berususan dengan directed graphs yang dibangun dari vertices (simpul) dan edges (sisi). Setelah membangun graph, algoritma-algoritma di sana akan membantu menemukan, misalnya jalur terpendek dari 2 simpul (titik) atau perulangan di graph.

Diberikan 3 simpul (vertices), temukan jalur terpendek dari simpul pertama ke terakhir.

```elixir
disgraph = :disgraph.new()

coords = [{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]

[v0, v1, v3] = (for c <- coords, do: :disgraph.add_vertex(disgraph, c))

:digraph.add_edge(digraph, v0, v1)
:digraph.add_edge(digraph, v1, v2)
:digraph.get_short_path(digraph, v0, v2)

[{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
```

Perlu diperhatikan bahwa function di `:digraph` menggantikan struktur graph di tmepat (in-place), ini memungkinkan karena mereka diimplementasikan sebagai table ETS, seperti dijelaskan berikutnya.

**Erlang Term Storage**
module `:ets` dan `:dets` menghandle storage dari data struktur yang besar di memory atau di disk.

ETS memungkinkan untuk membuat sebuah table yang berisi tuple. by default, ETS tables protected, yang berarti hanya owner dari process yang bisa menulis ke table tapi process lainnya dapat membaca. ETC mempunyai beberapa functionality untuk mengijinkan sebuah table untuk digunakan sebagai database sederhana, sebuah key-value store atau sebagai cache mechanism.

Function di `ets` module akan memodifikasi state dari table sebagai side-effect.

```elixir
table = :ets.new(:ets_test, [])
# Store as tuples dengan {name, population}

:ets.insert(table, {"China", 1_374_000_000})
:ets.insert(table, {"India", 1_284_000_000})
:ets.insert(table, {"USA", 322_000_000})
:ets.i(table)
<1   > {<<"India">>,1284000000}
<2   > {<<"USA">>,322000000}
<3   > {<<"China">>,1374000000}
```

**The math module**
`:math` module berisi operasi matematika yang mengcover trigonometry, exponential, dan function logarithmic.

```elixir
angle_45_deg = :math.pi() * 45.0 / 180.0

:math.sin(angle_45_deg)
0.7071067811865475
:math.exp(55.0)
7.694785265142018e23
:math.log(7.694785265142018e23)
55.0
```

**The queue module**
`:queue` module menyediakan sebuah data struktur yang mengimplement (double-ended) FIFO (first-in first-out) queue secara efisien:

```elixir
q = :queue.new
q = :queue.in("A", q)
q = :queue.in("B", q)
{value, q} = :queue.out(q)
value
{:value, "A"}
{value, q} = :queue.out(q)
value
{:value, "B"}
{value, q} = :queue.out(q)
value
:empty
```

**The rand module**

`:rand` mempunya function untuk mengembalikan random values dan mengatur random seed.

```elixir
:rand.uniform()
0.8175669086010815

_ = :rand.seed(:exs1024, {123, 123534, 345345})

:rand.uniform()
0.5820506340260994

:rand.uniform(6)
6
```

**The zip and zlib modules**
`:zip` module mengijinkan untuk membaca dan menulis ZIP files ke dan dari disk atau memory, begitu juga mengexstract informasi file.

Kode berikut digunakan untuk menghitung jumlah files di sebuah ZIP file:

```elixir
:zip.foldl(fn _, _, _, acc -> acc + 1 end, 0, :binary.bin_to_list("file.zip"))

{:ok, 633}
```

`:zlib` module berurusan dengan data compression di zlib format, seperti ditemukan juga di `gzip` command line utility ditemukan di Unix systems.

```elixir
song = "
Mary had a little lamb,
His fleece was white as snow,
And everywhere that Mary went,
The lamb was sure to go."
compressed = :zlib.compress(song)
byte_size(song)
110
byte_size(compressed)
99
:zlib.uncompress(compressed)
"\nMary had a little lamb,\nHis fleece was white as snow,\nAnd everywhere that Mary went,\nThe lamb was sure to go."
```

**Learning Erlang**
Jika mau belajar Erlang lebih dalam, ini adalah daftar online resources yang mengcover Erlang's fundamentals dan fitur yang lebih advanced:

- [Erlang crash course](https://elixir-lang.org/crash-course.html). menyediakan intro ke Erlang's syntax. Masing-masing cuplikan kode dibersamai dengan kode yang sama di Elixir. Ini adalah sebuah kesempatan untukmu untuk tidak hanya mendapatkan Exposure ke Erlang syntax tapi juga mereview apa yang dipelajari tentang Elixir

- Erlang official website mempunyai [tutorial singkat](https://www.erlang.org/course). Ada bab yang mendeskripsikan Erlang primitif untuk [concurrent programming](https://www.erlang.org/course/concurrent_programming.html)

- [Lean You Some Erlang for Great Good!](http://learnyousomeerlang.com/) adalah pengenalan yang baik ke Erlang, Ada design principles, standard library, best practices, dan banyak lagi. Sekali saja sudah membaca tutorial di atas, kamu bisa skip aja bagian pertama dari bab yang berurusan dengan syntax. Ketika sudah mencapai bab [The Hitchhiker's Guide to Concurrency](http://learnyousomeerlang.com/the-hitchhikers-guide-to-concurrency), the real fun baru mulai.

Langkah terakhir untuk dilihat di Elixir (dan Erlang) libraries yang mungkin digunakan ketika debugging.
