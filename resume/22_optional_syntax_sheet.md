## Optional syntax sheet

Pada panduan ini, kita belajar bahwa Elixir syntax mengijinkan developer untuk menghilangkan delimiter di beberapa kesempatan untuk membuat kode lebih mudah dibaca. Sebagai contoh, kita belajar kalo tanda kurung (parentheses) adalah opsional:

```elixir
length([1,2,3]) == length [1,2,3]
true
```

dan blok `do` - `end` sama dengan keyword list:

```elixir
# do-end blocks

if true do
  :this
else
  :that
end

:this

# keyword lists
if true, do: :this, else: :that

:this
```

Keyword list menggunakan Elixir regular notation untuk argument terpisah, di mana kita memisahkan masing-masing key-value pair dengan koma, dan masing-masing key diikuti dengan `:`. Di block `do`, kita menghapus tanda titik dua (:), koma dan memisahkan masing-masing keyword dengan garis baru. Mereka sangat berguna kareana mereka menghapus keterangan-keterangan ketika menulis blok dari kode. Sering kali kita menggunakan syntaks blok, tapi ini bagus untuk tahu bahwa mereka sama.

Untuk kenyamanan itu, yang mana kita panggil di sini "optional syntax", mengijinkan bahasa syntak core untuk lebih kecil, tanpa mengorbankan readibility dan ekspresifnya dari kodemu. Di bab ini, kita akan mereview 4 atusan yang disediakan oleh Elixir, menggunakan short snippet sebagai playground:

**Walk-through**
Ambil kode ini:

```elixir
if variable? do
  Call.this()
else
  Call.that()
end
```

Sekarang, hapus satu per satu kenyamanannya (conveniences):

1. `do` - `end` sama dengan keywords:

```elixir
if variable?, do: Call.this(), else: Call.that()
```

2. Keyword list sebagai argument terakhir tidak diperlukan square bracket (`[]`), tapi mari tambahkan mereka:

```elixir
if variable?, [do: Call.this(), else: Call.that()]
```

3. Keyword list sama dengan list dari dua-element tupples:

```elixir
if variable?, [{:do, Call.this()}, {:else, Call.that()}]
```

4. Akhirnya, parentheses (tanda kurung) adalah opsional di function call, tapi mari tambahkan saja:

```elixir
if(variable?), [{:do, Call.this()}, {:else, Call.that()}]
```

itu saja, 4 aturan outlined optinal syntax yang tersedia di Elixir

Untuk mengerti kenapa aturan ini penting, kita dapat membandingkan Elixir dnegan banyak bahasa pemrograman lainnya. Banyak bahasa pemrograman mempunyai banyak keyword untuk mendefinisikan method, function, conditional, loops, dan lainnya.masing-masing keyword memiliki syntaks mereka masing-masing yang ditautkan ke mereka.

Bagaimanpun, di Elixir, tidak ada satupun dari fitur-fitur bahasa ini yang membutuhkan "keyword" khusus, melainkan semuanya dibangun dari sekumpulan aturan kecil ini. Manfaat lainnya adalah developer juga dapat memperluas bahasa dengan cara yang konsisten dengan bahasa itu sendiri, kareana konstruksi untuk mendesain dan memperluas bahasa adalah sama. Kita akan membahasanya lebih lanjut dalam panduan [Meta-programming](https://hexdocs.pm/elixir/1.16/quote-and-unquote.html).

Pada akhirnya, aturan itu adalah apa yang memungkinkan untuk menulis:

```elixir
defmodule Math do
  def add(a, b) do
    a + b
  end
end
```

daripada ini:

```elixir
defmodule(Math, [
    {:do, def(add(a, b), [{:do, a + b}])}
])
```

Kapanpun kamu mempunyai pertanyaan, perjalan singkat ini telah membahasnya.

Akhirnya, jika kamu punya concern tentang kapan untuk mengaplikasikan aturan ini, Peting untuk tahu bahwa Elixir formatter akan menghandlenya. Banyak Elixir developer menggunakan `mix format` untuk memformat codebase mereka berdasarkan well-defined aturan yang didefinisikan oleh Elixir team dan community. Sebagai contoh, `mix format` akan selalu menambahkan parentheses (tanda kurung) untuk function call kecuali secara explisit mengkonfigurasi untuk tidak melakukannya. Ini membantu untuk maintenance lintas codebase di dalam organisasi dan komunitas yang lebih luas.
