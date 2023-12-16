## Writing documentation

Elixir memperlakukan documentation sebagai first-class citizen. Documentation harus mudah ditulis dan dibaca. Di sini akan kita pelajari cara menulis documentation di Elixir, mencover construct seperti modul attribut, style practices dan doctests.

**Markdown**

Elixir documentation ditulis menggunakan markdown. Ada banyak panduan untuk menulis markdown, kita rekomendasikan satu dari github sebagai titik awal:

- [Basic writing and formatting syntax](https://help.github.com/articles/basic-writing-and-formatting-syntax/)

\*\*Module Attributes
Documentation di elixir biasanya ditempelkan ke module attributes. Berikut contohnya:

```elixir
defmodule MyApp.Hello do
  @moduledoc """
  This is the hello module
  """
  @moduledoc since: "1.0.0"

  @doc """
  Says hello to the given 'name'

  Return `:ok`.

  ## Examples

    iex> MyApp.Hello.world(:john)
    :ok


  """
  @doc since: "1.3.0"
  def world(name) do
    IO.puts("hello #{name}")
  end
end
```

`@moduledoc` attributes digunakan untuk menambahkan documentation ke module. `@doc` digunakan sebelum sebuah function untuk menyediakan documentation terhadap function tersebut. Di samping attributes di atas, `@typedoc` dapat juga digunakan untuk menempelkan documentation ke type yang didefinisikan sebagai bagian dari typespecs, yang akan kita explore nanti. Elixir memungkinkan metadata untuk di tempelkan ke documentation, dengan meneruskan sebuah keyword ke `@doc` dan teman-temannya.

**Function Arguments**
Ketika mendokumentasikan sebuah function, nama argumen disimpulkan oleh compiler. Sebagai contoh:

```elixir
def size(%{size: size}) do
  size
end
```

Compiler akan menyimpulkan argumen ini sebagai map. kadang-kadang inference tidak optimal, terutama jika function berisi beberapa klausa dengan pencocokan argument pada nilai yang berbeda setiap kali. Kamu dapat menentukan nama yang tepat untuk dokumentasi dengan mendeklarasikan hanya function head kapan saja sebelum implementasi:

```elixir
def size(map_with_size)
def size(%{size: size}) do
  size
end
```

**Documentation metadata**
Elixir memungkinkan developer untuk melampirkan metadata sembarang (arbitrary) ke dalam dokumentasi. Hal ini dilakukan dengan mengoper sebuah keyword list ke attribut yang relevan(seperti `@moduledoc`, `@typedoc`, dan `@doc`). Metadata yang umum digunakan adalah `:since` yang memberi keterangan pada versi berapa module, function, type atau callback tertentu ditambahkan, seperti ditunjukkan di contoh di atas.

Metadata yang umum lainnya adalah `:deprecated`, yang memancarkan sebuah warning di dokumentasi, menjelaskan bahwa penggunaaannya itu tidak disarankan:

```elixir
@doc deprecated: "Use Foo.bar/2 instead"
```

Perhatikan bahawa `:deprecated` key tidak memperingatkan ketika seorang developer memanggil function. Jika kamu ingin kode juga memancarkan sebuah warning, kamu dapat menggunakan `@deprecated` attribute:

```elixir
@deprecated "Use Foo.bar/2 instead"
```

Metadata dapat mempunya kunci apapun. Dokumentasi tool sering kali menggunakan metadata untuk menyediakan data lebih ke pembaca dan untuk memperkaya user experience.

**Recomendations**

Ketika menulis dokumentasi:

- Buatlah paragraf pertama dari dokumentasi menjadi ringkas dan sederhana, biasanya satu baris. Alat bantu seperti [ExDoc](https://github.com/elixir-lang/ex_doc/) menggunakan baris pertama untuk membuat ringkasan.
- Rujuk module dengan nama mereka.

Markdown menggunakan tanda petik () untuk mengutip kode. Elixir dibangun di atas hal tersebut untuk seara otomatis membuat tautan ketika nama modul atau function direferensikan. Untuk alasan ini, selalu gunakan nama modul secara lengkap. Jika anda memiliki modul dengan nama `MyApp.Hello`, selalu rujuk modul sebagai `MyApp.Hello` dan jangan pernah sebagai `Hello`.

- Rujuk function dengan nama dan arity jika mereka adalah local, seperti di `world/1`, atau dengan modul, nama dan arity jika mengarah ke modul external: `MyApp.Hello.world/1`.
- Rujuk sebuah `@callback` dengan awalan `c:`, seperti di `c:world/1`
- Rujuak sebuah `@type` dengan awalan `t:`, seperti `t:values/0`
- Mulai sesi baru dengan headers level kedua Markdown `##`. Header level pertama sudah dipesan untuk module dan function name.
- Letakkan dokumentasi sebelum klausa pertama dari function banyak-klausa (multi-clause).
  Dokumentasi selalu per function dan arity dan tidak per klausa.
- Gunakan key `:since` di dokumentasi metadata untuk membuat anotasi setiap function atau modul baru ditambahkan ke API anda.

**Doctests**
Kita merekomendasikan developer menyertakan contoh di dokumentasi mereka, sering di bawah heading `## Example` mereka sendiri. Untuk memastikan contoh tidak kadaluarsa, Framework test Elixir (ExUnit) menyediakan fitur untuk memanggil doctests yang memungkinkan developer untuk mengetes contoh di dokumentasi mereka. Doctest bekerja dengan cara memparsing kode sample dimulai dengan `iex>` dari dokumentasi. kamu dapat membacanya di [ExUnit.DocTest](https://hexdocs.pm/ex_unit/ExUnit.DocTest.html)

**Documentation != Code comments**
Elixir memperlakukan dokumentasi dan code comment sebagai konsep yang berbeda. Dokumentasi adalah sebuah kontrak explisit antara anda dan pengguna dari Application Programming Interface (API), menjadikan mereka developer pihak ketiga, co-workers, atau dirimu di masa depan. Module dan function harus selalu didokumentasikan jika mereka adalah bagian dari API-mu.

Code comment ditujukan untuk developer membaca kode. Mereka sangat berguna untuk menandai peningkatan, meninggalkan notes (sebagai contoh, kenapa kamu harus menggunakan solusi karena adanya bug di libraries), dan seterusnya. Mereka terikat ke source code: Kamu dapat sepenuhnya menulis sebuah function dan menghapus semua code comment yang ada, dan itu akan berperilaku sama, tanpa perubahan pada perilaku atau dokumentasinya.

Karena private function tidak dapat diakses secara external, Elixir akan memperingatkan jika sebuah private function mempunyai attribut `@doc` dan akan mengabaikan kontennya. Bagaimanpun, kamu dapat menambahkan code comment untuk private functions, seperti dengan kode lainnya, dan kita merekomendasikan developer untuk melakukannya juga kapanpun mereka percaya itu akan menambahkan informasi yang relevan untuk pembaca dan maintainer dari kode tersebut.

Singkatnya, dokumentasi adalah kontrak antara pengguna API-mu, siapa yang tidak perlu memiliki akses ke source code, sedangkan code comment adalah untuk siapa saja yang berinteraksi secara langsung dengan source code. Kamu dapat mempelajari dan mengekpresikan perbedaaan jaminaan tentang softwaremu dengan memisahkan 2 konsep itu.

**Hiding internal Module dan Function**
Selain module dan function libraries yang disediakan sebagai bagian dari public interfacenya, libary juga mengimplement functionality penting yang tidak bagian dari API. Meskipun module dan function itu dapat diakses, module dan function ini dimaksudkan untuk internal library dan dengan demikian harus memiliki dokumentasi untuk end user.

Dengan mudahnya, Elixir memungkinkan developer untuk menyembunyikan module dan function dari dokumentasi, dengan menyetting `@doc false` untuk menyembunyikan function khusus, atau `@moduledoc false` untuk menyembunyikan seluruh module. Jika module disembunyikan, kamu bahkan dapat mendokumentasikan function-function dalam modul tersebut, tapi modul itu sendiri tidak akan terdaftar dalam dokumentasi:

```elixir
defmodule MyApp.Hidden do
  @moduledoc false

  @doc """
  This function won't be listed in docs
  """
  def function_that_wont_be_listed_in_docs do
    # ...
  end
end
```

Dalam kasus kamu tidak ingin menyembunyikan seluruh modul,kamu dapat menyembunyikan function secara individual:

```elixir
defmodule MyApp.Sample do
  @doc false
  def add(a, b), do: a + b
end

```

Bagaimanapun, perlu diingat `@moduledoc false` atau `@doc false` tidak membuat sebuah function private. Function di atas dapat tetap dipanggil sebagai `MyApp.Sample.add(1, 2)`. Tidak hanya itu, jika `MyApp.Sample` diimport, function `add/2` akan juga diimport ke pemanggilnya. Untuk alasan itu, berhati-hatilah saat menambahkan `@doc false` ke function, sebagai gantinya gunakan salah satu dari dua opsi ini:

- Pindahkan function tidak terdokumentasi ke sebuah modul dengan `@moduledoc false`, seperti `MyApp.Hidden`, memastikan function tidak secara sengaja terexpose atau diimport. Ingat bahwa kamu dapat menggunakan `@moduledoc` untuk menyembunyikan seluruh modul dan tetep mendokumentasikan masing-masing function dengan `@doc`. Alat ini tetap mengabaikan modul.
- Awali function name dengan satu atau dua underscores, sebagai contoh, `__add__/2`. Function yang dimulai dengan underscore secara otomatis diperlakukan sebagai disembunyikan (hidden), meskipun kamu dapat juga membuat function tersebut eksplisit dan menambhkan `@doc false`. Compiler tidak akan mengimport function yang dimulai dengan underscore dan function-function tersebut mengisyaratkan kepada siapapun yang membaca kode bahwa function tersebut digunakan secara pribadi.

Elixir menyimpan dokumentasi di dalam potongan-potongan yang sudah ditentukan di dalam bytecode. Dokumentasi tidak dimuat ke dalam memori ketika module di load, namun, dokumentasi dapat dibaca dari bytecode di dalam disk menggunakan function `Code.fetch_docs/1`. Kelemahannya adalah modul yang didefinisikan di dalam memori, seperti yang didefinisikan di `IEx`, tidak dapat diakse dokumentasinya karena tidak menulis bycode ke disk.
