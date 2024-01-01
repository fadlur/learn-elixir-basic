## Debugging

Ada banyak cara untuk debug di Elixir, di bab ini akan kita bahas beberapa dari cara yang umum digunakan.

**IO.inspect/2**
Yang membuat `IO.inspect(item, opts \\ [])` berguna untuk debugging adalah karena mengembalikan `item` argument yang dimasukkan tanpa mempengaruhi perilaku dari original code. Bisa dilihat contoh berikut:

```elixir
(1..10)
|> IO.inspect()
|> Enum.map(fn x -> x * 2 end)
|> IO.inspect()
|> Enum.sum()
|> IO.inspect()
```

Hasilnya di terminal.

```elixir
1..10
[2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
110
```

Seperti yang dapat kita lihat `IO.inspect/2` memungkinkan untuk "memata-matai" nilai dari hampir semua di dalam kode tanpa menganti hasilnya, membuatnya sangat membantu di dalam pipeline seperti kasus di atas.

`IO.inspect/2` juga menyediakan kemampuan untuk mendekorasi output dengan `label` option. Label akan dicetak sebelum `item` yang diinspect:

```elixir
[1,2,3]
|> IO.inspect(label: "before")
|> Enum.map(&(&1 * 2))
|> IO.inspect(label: "after")
|> Enum.sum
```

hasilnya di terminal:

```elixir
before: [1, 2, 3]
after: [2, 4, 6]
```

Umum juga menggunakan `IO.inspect/2` dengan `binding/0`, yang mengembalikan semua nama variable dan nilai mereka:

```elixir
def some_fun(a, b, c) do
  IO.inspect binding()
  ...
end
```

Ketika `some_fun/3` dipanggil dengan `:foo`, `"bar"`, `:baz` maka akan mencetak:

```elixir
[a: :foo, b: "bar", c: :baz]
```

Buka halaman [IO.inspect/2](https://hexdocs.pm/elixir/1.16/IO.html#inspect/2) dan [Inspect.Opts](https://hexdocs.pm/elixir/1.16/Inspect.Opts.html) untuk belajar lebih lanjut tentang function dan option yang disupport.

**dbg/2**

Elixir v1.14 memperkenalkan `dbg/2`. sama saja dengan `IO.inspect/2` tapi secara khusus dibuat untuk debugging. `dbg/2` akan mencetak value yang diberikan dan mengembalikannya (sama dengan `IO.inspect/2`), tapi function tersebut hanya mencetak kode dan lokasinya.

```elixir
# In my_file.exs

feature = %{name: :dbg, inspiration: "Rust"}
dbg(feature)
dbg(Map.put(feature, :in_version, "1.14.0"))
```

Kode di atas akan mencetak ini:

```elixir
[my_file.exs:2: (file)]
feature #=> %{inspiration: "Rust", name: :dbg}
[my_file.exs:3: (file)]
Map.put(feature, :in_version, "1.14.0") #=> %{in_version: "1.14.0", inspiration: "Rust", name: :dbg}
```

Ketika membicarakan `IO.inspect/2`, kita menyebut kegunaannya ketika ditempatkan di antara `|>` pipelines. `dbg` melakukannya dengan lebih baik. function itu mengerti kode Elixir, sehingga akan mencetak values di tiap langkah dari pipeline.

```elixir
# In dbg_pipes.exs
__ENV__.file
|> String.split("/", trim: true)
|> List.last()
|> File.exists?()
|> dbg()
```

Kode ini akan mencetak:

```elixir
[dbg_pipes.exs:5: (file)]
__ENV__.file #=> "/home/myuser/dbg_pipes.exs"
|> String.split("/", trim: true) #=> ["home", "myuser", "dbg_pipes.exs"]
|> List.last() #=> "dbg_pipes.exs"
|> File.exists?() #=> true
```

Ketika `dbg` menyediakan kenyamanan di konstruksi Elixir, kamu akan membutuhkan `IEx` jika kamu ingin mengeksekusi kode dan mengatur breakpoints ketika debugging.

**Breakpoints**
Ketika menggunakan `IEx`, kamu mungkin harus memberikan `--dbg pry` sebagai sebuah option untuk "menghentikan" eksekusi kode di mana `dbg` dipanggil:

```elixir
iex --dbg pry
```

Sekarang panggilan ke `dbg` akan bertanya jika kamu ingin membongkar (pry) existing code. Jika kamu menerima (ok atau accept), kamu akan dapat mengakses semua variable, begitu juga import dan alias dari kode, langsung dari IEx. Ini dipanggil "prying", Ketika pry session berjalan, eksekusi kode berhenti, sampai `continue` atau `next` dipanggil. Ingat kamu dapat selalu menjalankan `iex` di dalam context sebuah project dengan `iex -S mix TASK`.

<script id="asciicast-509509" src="https://asciinema.org/a/509509.js" async></script>

`dbg` memerlukan kita untuk mengubah kode yang mau kita debug dan mempunyai functionality yang terbatas. Untuknya IEx juga menyediakan `IEx.break!/2` yang mengijinkan kita untuk mengatur dan memanage breakpoint di kode Elixir manapun tanpa memodifikasi aslinya:

<script type="text/javascript" src="https://asciinema.org/a/0h3po0AmTcBAorc5GBNU97nrs.js" id="asciicast-0h3po0AmTcBAorc5GBNU97nrs" async></script>

Sama dengan `dbg`, saat breakpoint dicapai, eksekusi kode akan berhenti sampai `continue` atau `next` dipanggil. Bagaimanapun, `break!/2` tidak mempunya akses ke alias dan import dari kode yang didebug seperti saat berjalan di artifact yang dikompilasi dibandingkan di source code.

**Observer**
Untuk debugging system yang kompleks, melompat2 di kode tidaklah cukup perlu mempunyai pemahaman keseluruhan virtual machine, processes, applications, seperti mengatur mekanisme tracing. Untuk ini dapat dicapai di Erlang dengan `:observer`.
Di aplikasimu:

```elixir
iex
:observer.start()
```

Kode di atas akan membuka GUI yang menyediakan banyak panel untuk mengerti dan navigasi runtime dan projectmu.

Kita akan explore lebih dalam di context dari sebuah project beneran di [the Dynamic Supervisor chapter of the Mix & OTP guide](https://hexdocs.pm/elixir/1.16/dynamic-supervisor.html). Ini adalah salah satu dari teknik debugging [the Phoenix framework used to achieve 2 million connection on a single machine](https://phoenixframework.org/blog/the-road-to-2-million-websocket-connections)

Jika menggunakna phoenix framework, Observer ada di [Phoenix LiveDashboard](https://github.com/phoenixframework/phoenix_live_dashboard). sebuah web dashboard untuk production nodes yang menyediakan fitur yang sama dengan observer.

Akhirnya, ingat kamu juga dapat sebuah mini-overview dari runtime info dengan memanggil `runtime_info/0` secara langsung di IEx.

**Other tools and community**

Kita hanya baru mengulas permukaan dari apa yang ditawarkan oleh Erlang VM, sebagai contoh:

- Bersama dengan observer application, Erlang juga menyertakan sebuah [:crashdump_viewer](https://www.erlang.org/doc/man/crashdump_viewer.html) untuk melihat crash dumps.
- Integration with OS level tracers, seperti [Linux Trace Toolkit](http://www.erlang.org/doc/apps/runtime_tools/LTTng.html), [DTRACE](http://www.erlang.org/doc/apps/runtime_tools/DTRACE.html) dan [SystemTap](http://www.erlang.org/doc/apps/runtime_tools/SYSTEMTAP.html)
- [Microstate accounting](http://www.erlang.org/doc/man/msacc.html) mengukur berapa banyak waktu runtim dihabiskan di beberapa low-level task di interval waktu yang singkat.
- Mix berisi banyak task di bawah namespace `profile`, seperti `cprof` dan `fprof`
- Untuk use case yang lebih advance, kita merekomendasikan [Erlang in Anger](https://www.erlang-in-anger.com/) - ini free lho.
