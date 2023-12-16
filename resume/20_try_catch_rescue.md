## try, catch, and rescue

Elixir mempunya 3 mekanisme error: errors, throws, dan exits. Di bab ini, kita akan mengeksplore masing-masing dan menyertakan keterangan ketika masing-masing digunakan.

**Errors**
Errors (atau exceptions) digunakan ketika exceptional terjadi di kode.
Sebagai contoh, error dapat ditampikan ketika menambahkan angka ke atom:

```elixir
:foo + 1
** (ArithmeticError) bad argument in arithmetic expression
    :erlang.+(:foo, 1)
```

Sebuah runtime error bisa ditampilkan kapan saja dengan menggunakan `raise/1`:

```elixir
raise "oops"
** (RuntimeError) oops
```

Error lainnya dapat dimunculkan dengan `raise/2` dengan menambahkan error name dan sebuah list dari keyword argument:

```elixir
raise ArgumentError, message: "invalid argument too"
** (ArgumentError) invalid argument foo
```

Kamu dapat juga mendefinisikan errormu sendiri dengan membuat sebuah modul dan dengan menggunakan `defexception/1` construct di dalamnya. Cara ini, kamu akan mmebuat sebuah error dengan nama yang sama dengan modul yang didefinisikan di dalamnya. Kasus paling umum adalah mendefinisikan custom exception dengan sebuah message field:

```elixir
defmodule MyError do
  defexception message: "default message"
end
raise MyError
** (MyError) default message
raise MyError, message: "custom message"
** (MyError) custom message
```

Errors dapat di rescue menggunakan `try/rescue` construct:

```elixir
try do
  raise "oops"
rescue
  e in RuntimeError -> e
end
%RuntimeError{message: "oops"}
```

Pada contoh di atas, rescue the runtime error dan mengembalikan exception sendiri, yang kemudian diprint di `iex` session.

Jika kamu tidak menggunakan exception, Kamu tidak perlu mengoper variabel untuk `rescue`:

```elixir
try do
  raise "oops"
rescue
  RuntimeError -> "Error!"
end
"Error!"
```

Pada praktiknya, Developer elixir jarang menggunakan `try/rescue` construct. Sebagai contoh, banyak bahasa akan memaksa kamu untuk rescue sebuah error ketika sebuah file tidak dapat dibuka. Elixir malah menyediakan `File.read/1` yang mengembalikan sebuah tuple yang berisi informasi apakah file tersebut berhasil dibuka:

```elixir
File.read("hello")
{:error, :enoent}
File.write("hello", "world")
:ok
File.read("hello")
{:ok, "world"}
```

Tidak ada `try/rescue` di sini. Jika kamu mau menghandle banyak keluaran dari sebuah file, kamu dapat menggunakan pattern matching menggunakan `case` construct:

```elixir
case File.read("hello") do
  {:ok, body} -> IO.puts("Success: #{body}")
  {:error, reason} -> IO.puts("Error: #{reason}")
end
```

Untuk kasus di mana kamu mengharapkan sebuah file ada (dan jika tidak ada file adalah bener-bener sebuah error) kamu bisa menggunakan `File.read!/1`:

```elixir
File.read!("unknown")
** (File.Error) could not read file "unknown": no such file or directory
    (elixir) lib/file.ex:272: File.read!/1
```

Pada akhirnya, itu terserah aplikasimu yang menentukan jika sebuah error ketika membuka sebuah file adalah exceptional atau tidak. Itulah mengapa Elixir tidak memaksa exception di `File.read/1` dan banyak lagi function lainnya. Malahan, menyerahkan kepada developer untuk memilih mana cara yang terbaik untuk diproses.

Banyak function di standard library mengikuti pattern/pola yang memunculkan exception dibanding mengembalikan tuple untuk dicocokkan. Konveksi ini digunakan untuk membuat sebuah function (`foo`) yang mengembalikan tuple `{:ok, result}` atau `{:error, reason}` dan function lainnya (`foo!` function dengan nama yang sama tapi dengan tanda seru `!`) yang mengambil argument yang sama seperti `foo` tapi yang memunculkan error kalo ada error. `foo!` harus mengembalikan sebuah hasil (tidak dibungkus di tuple) jika semuanya berjalan dengan baik. Module `File` adalah contoh yang bagus tenant konvensi ini.

**Fail fast/Let it crash**
Seseorang berkata bahwa umum di Erlang community, seperti elixir, jika "fail fast" / "let it crash". Ide dibalik let it crash adalah ketika dalam kasus terjadi sesuatu yang tidak diinginkan. Yang terbaik adalah membiarkan exception terjadi, tanpa menyelamatkannya.

Penting untuk menegaskan kata unexpected. contoh, bayangkan kamu sedang membuat sebuah script untuk process file. Scriptmu menerima sebuah filename sebagai input. Script itu mengharapkan bahwa user mungkin membuat kesalah dan memberi nama yang tidak dikenal. Di skenario ini, ketika kamu dapat menggunakan `File.read!/1` untuk membaca file dan membiarkannya crash dalam kasus filenamenya invalid, Itu mungkin masuk akal untuk menggunakan `File.read/1` dan memberikan pengguna scriptmu dengan sebuah feedback yang presisi dan jelas tentang apa yang akan terjadi.

Di lain waktu, kamu mungkin mengharapkan file tertentu ada, dan di kasus itu tidak ada, itu mungkin sesuai terjadi di suatu tempat. Di kasus itu `File.read!/1` adalah yang kamu butuhkan.

Pendekatan kedua juga mungkin bekerja, seperti didiskusikan di bab Process, semua kode Elixir berjalan di dalam proses adalah terisolasi dan tidak membagi apapun secara default. Karena itu, sebuah exception yang tidak tertangani di sebuah process tidak akan crash atau mengkorupsi state dari proses lainnya. Ini mengijinkan kita untuk mendefinisikan process supervisor, yang berarti mengobservasi kapan sebuah process dimatikan tiba-tiba, dan memulai sebuah process supervisor di tempat itu.

Pada akhirnya, "fail fast" / "let it crash" adalah cara untuk mengatakan itu, ketika sesuatu yang tidak diharapakan terjadi, maka cara terbaik adalah memulai dari awal di dalam sebuah process baru, baru saja dimulai oleh supervisor, dibandingkan meraba-raba sambil mencoba menyelamatkan semua prosess tanpa tahu konteks kapan dan bagimana mereke dapat terjadi.

**Reraise**
Ketika kita umumnya menghindari `try/rescue` di Elixir, satu situasi di mana kita mungkin ingin menggunakan construct adalah untuk observability/monitoring. Bayangkan kamu ingin meng-log kalau sesuatu yang salah terjadi, kamu dapat melakukan:

```elixir
try do
  .... some code ....
rescue
  e ->
    Logger.error(Exception.format(:error, e, __STACKTRACE__))
    reraise e, __STACKTRACE__
```

Di contoh di atas, kita merescue exception, logged it, dan kemudian re-raised it. Kita menggunakan `__STACKTRACE__` construct berdua ketaika memformat exception dan kapan re-raising. Ini meyakinkan kita reraise exception seperti seharusnya, tanpa mengubah value atau asalnya.

Umumnya, kita mengambil errors di Elixir secara literal: Mereka reserved untuk situasi exceptional dan/atau tidak diharapkan. Tidak pernah mengontrol aliran kode kita. Di kasus kamu secara aktual membutuhkan aliran control konstruks, throws harus digunakan. Itu yang akan kita lihat selanjutnya.

**Throws**
Di Elixir, sebuah value dapat dilemparkan (thrown) dan ditangkap (caught) kemudian. `throw` dan `catch` diperuntukkan untuk situasi dimana kondisi itu tidak memungkinkan untuk menerima sebuah value kecuali menggunakan `throw` dan `catch`

Situasi itu sangat tidak umum pada praktiknya kecuali ketika interfacing dengan library yang tidak menyediakan API yang proper. Untuk contoh, mari bayangkan module `Enum` tidak menyediakan API apapun untuk menemukan sebuah value dan bahwa kita perlu menemukan kelipatan pertama dari 13 dalam sebuah list angka:

```elixir
try do
  Enum.each(-50..50, fn x ->
    if rem(x, 13) == 0, do: throw(x)
  end)
  "Got nothing"
catch
  x -> "Got #{x}"
end
"Got -39"
```

Sejak `Enum` tidak menyediakan API yang proper, maka `Enum.find/2` adalah cara yang bisa dipakai:

```elixir
Enum.find(-50..50, &(rem(&1, 13) == 0))
```

**Exits**
Seluruh kode elixir berjalan di dalam proses yang berkomunikasi satu sama lain. Ketika sebuah process mati karena "sebab yang normal" (e.g, unhandled exceptions), process itu mengirim singal `exit`. Sebuah process dapat juga mati dengan secara explisit mengirim sinyal `exit`:

```elixir
spawn_link(fn -> exit(1) end)
** (EXIT from #PID<0.56.0>) shell process exited with reason: 1
```

Pada contoh di atas, linked process mati dengan mengirim sebuah signal `exit` dengan sebuah nilai = 1. Elixir shell secara otomatis menghandle pesan itu dan mencetaknya ke terminal.

`exit` dapat juga "ditangkap (caught)" menggunakan `try/catch`:

```elixir
try do
  exit("I am exiting")
catch
  :exit, _ -> "not really"
end
```

menggunakan `try/catch` sudah tidak umum dan menggunakan untuk menangkap exit malah lebih jarang lagi.

sinyal `exit` adalah bagian penting dari fault tolerant system yang disediakan oleh Erlang VM. Process biasanya berjalan di bawah supervision trees yang mereka sendiri mendengarkan (listen) sinyal `exit` dari process yang disupervised. Sekali sebuah sinyal `exit` diterima, strategi supervision dinyalakan dan process supervised direstart.

Supervision system ini yang membuat konstruksi (construct) seperti `try/catch` dan `try/rescue` menjadi tidak umum di Elixir. Daripada menyelamatkan (rescue) sebuah error, kita lebih suka "fail fast/gagal dengan cepat" karena supervision tree menggaranksi aplikasi akan kembali lagi ke kondisi awal yang diketahui setelah error terjadi.

**After**
Kadang-kadang perlu untuk memastikan bahwa sebuah resource dibersihkan setelah beberapa action yang mempunyai potensi memunculkan error. `try/after` construct mengijinkanmu untuk melakukan itu. Sebagai contoh, kita dapat membuat sebuahfile dan menggunakan sebuah `after` clasuse untuk menutupnya. bahkan jika terjadi sesuatu yang salah.

```elixir
{:ok, file} = File.open("sample", [:utf8, :write])
try do
  IO.write(file, "olá")
  raise "oops, something went wrong"
after
  File.close(file)
end
** (RuntimeError) oops, something went wrong
```

`after` clause akan dieksekusi tidak peduli blok try berjalan sukses atau gagal. Perhatikan, bagaimanapun, jika sebuah linked process exits, process ini akan exit dan `after` clause tidak akan berjalan. Dengan demikian, `after` menyediakan hanya sebuah jaminan lunak. Untungnya, file-file di Elixir juga terhubung dengan process yang sedang berjalan dan oleh karena itu, file-file tersebut akan selalu ditutup jika process yang sedang berjalan itu mengalamai crash, tidak bergantung pada `after` clause. Kamu akan menemukan hal yang sama berlaku untuk resource lainnya seperti ETS table, socket, port dan lainnya.

kadang-kadang kamu mungkin ingin membungkus seluruh isi dari sebuah function dalam sebuah try construct, Sering kali untuk menjamin beberapa kode akan dieksekusi setelahnya. di kasus sperti ini, elixir mengijinkan kamu untuk menghilangkan baris `try`:

```elixir
defmodule RunAfter do
  def without_even_trying do
    raise "oops"
  after
    IO.puts "cleaning up!"
  end
end
RunAfter.without_even_trying
cleaning up!
** (RuntimeError) oops
```

Elixir akan secara otomatis membungkus function body di dalam sebuah `try` kapanpun satu dari `after`, `rescue` atau `catch` ditentukan.

**Else**
Jika sebuah blok `else` ada, blok itu akan cocok pada hasil dari blok `try` kapanpun blok `try` selesai tanpa _throw_ atau sebuah error:

```elixir
x = 2
2
try do
  1 / x
recue
 ArithmeticError ->
   :infinity
else
  y when y < 1 and y > -1 ->
    :small
  _ ->
    :large
end
:small
```

Exceptions di dalam blok `else` tidak ditangkap. Jika tidak ada pattern di dalam blok `else` cocok, sebuah exception akan dimunculkan. Exception ini tidak ditangkap oleh blok `try/catch/rescue/after` saat ini.

**variable scope**

Mirip dengan `case`, `cond`, `if` dan construct lainnya di Elixir, variable didefinisikan di dalam blok `try/catch/rescue/after` tidak akan bocor ke context di luarnya. Dengan kata lain, kode ini invalid:

```elixir
try do
  raise "fail"
  what_happened = :did_not_raise
rescue
  _ -> what_happened = :rescued
end
what_happened
** (CompileError) undefined variable "what_happened"
```

Malahan, kamu harus mengembalikan value dari `try` expression:

```elixir
what_happened =
  try do
    raise "fail"
    :did_not_raise
  rescue
    _ -> :rescued
  end
what_happened
:rescued
```

Lebih lanjut lagi, variable yang didefinisikan di blok "do" dari `try` tidak tersedia di dalam `rescue/after/else`. Ini karena blok `try` mungkin gagal kapanpun dan oleh karena itu variable-variablenya mungkin tidak pernah diikat (bound in) sejak awal. Jadi ini juga tidak valid:

```elixir
try do
  raise "fail"
  another_what_happened = :did_not_raise
rescue
  _ -> another_what_happened
end
** (CompileError) undefined variable "another_what_happened"
```

Ini mengakhiri perkenalaan kita di `try`, `catch` dan `rescue`. Kamu akan menemukan mereka jarang menggunakan di Elixir daripada di bahasa lainnya. kemudian kita akan berbicara tentang sebuah bahasan yang sangat penting untuk developer Elixir: writing documentation.
