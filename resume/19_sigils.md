## Sigils

Elixir menyediakan double-quotes strings seperti konsep charlist, yang didefinisikan menggunakan sintak sigil `~c"hello world"`.

Salah satu gol dari Elixir extensibility: developer bisa mengextends elixir untuk cocok di domain (bidang) apapun. Sigil menyediakan foundation untuk mengextend elixir dengan representasi textual custom. Sigil dimulai dengan karakter tilde (`~`) yang dimulai dengan satu huruf lower-case atau satu atau lebih upper-case, dan delimiter (pembatas). Optional modifier ditambahkan setelah delimiter (pembatas).

**Regular expression**
Sigil paling umum di elixir adalah `~r`, yang digunakan untuk membuat [regular expression](https://en.wikipedia.org/wiki/Regular_Expressions)

```elixir
# regular expresssion yang mencocokkan strings yang berisi "foo" atau "bar"
regex = ~/foo|bar/
~r/foo|bar/
"foo" =~ regex
true
"bat" =~ regex
false
```

Elixir menyediakan Perl-compatible regular expresssion (regexes), seperti diimplementasi oleh [PCRE](http://www.pcre.org/) library. Regexes juga mendukung modifiers. Sebagai contoh, `i` modifier membuat sebuah regular expression case sensitive:

```elixir
"HELLO" =~ ~r/hello/
false
"HELLO" =~ ~r/hello/i
true
```

Lihat modul [Regex](https://hexdocs.pm/elixir/1.16/Regex.html) untuk informasi lebih lanjut tentang modifier lainnya dan operasi yang didukung dengan regular expression.

Sejauh ini, semua contoh mempunya `/` untuk membatasi (delimit) sebuah regular expression. Bagaimanapun, sigils support 8 pembatas (delimiter):

```elixir
~/hello/
~|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>
```

Alasan dibalik mendukung delimiter berbeda adalah untuk menyediakan cara untuk menulis harfiah (literal) tanpa _escaping delimiter_. Sebagai contoh, sebuah regular expresssion dengan forward slashes seperti `~r(^https?://)` lebih enak dibaca dibanding `~/^https?:\/\//`. Sama halnya, jika regular expresssion mempunya forward slashes dan capturing grup (yang menggunakan `()`), Mungkin lebih memilih menggunakan double quote daripada parentheses (`()`)

**Strings, charlists,and word lists sigiles**
Di samping regular expresssions, Elixir mempunyai 3 sigils lainnya.

**Strings**
`~s` sigil digunakan untuk mengenerate strings seperti tanda petik dua. `~s` sigil sangat berguna ketika sebuah string terdiri double quotes (tanda petik dua)

```elixir
~s(this is a string with "double" quotes, not 'single' ones)
"this is a string with \"double\" quotes, not 'single' ones"
```

**Charlists**
`~c` sigil adalah cara umum untuk merepresentasi charlists.

```elixir
[?c, ?a, ?t]
~c"cat"
~c(this is a char list containing "double quotes")
~c"this is a char list containing \"double quotes\""
```

**Word lists**
`~w` sigil digunakan untuk men-generate lists dari kata (\_word (kata) hanyalah strings umum). Di dalam `~w` sigil, kata-kata dipisahkan dengan spasi.

```elixir
~w(foo bar bat)
["foo", "bar", "bat"]
```

`~w` sigil juga menerima `c`, `s`, `a` modifier untuk charlists, strings, dan atoms. Yang menentukan data type elemen dari list yang dihasilkan.

```elixir
~w(foo bar bat)a
[:foo, :bar, :bat]
```

**Interpolation and escaping in string sigils**
Elixir mendukung beberapa sigil varial untuk berurusan dengan escaping character dan interpolasi. Khususnya, sigil uppercase letters tidak melakukan interpolasi dan escaping. Sebagai contoh, meskipun `~s` dan `~S` akan menghasilkan strings, the former (yang pertama) mengijinkan escap code dan interpolasi ketika huruf tidak.

```elixir
~s(String with escape codes \x26 #{"inter" <> "polation"})
"String with escape codes & interpolation"
~S(String without escape codes \x26 without #{interpolation})
"String without escape codes \\x26 without \#{interpolation}"
```

Ecape code berikut dapat digunakan di strings dan charlists:

- `\\` – single backslash
- `\a` – bell/alert
- `\b` – backspace
- `\d` - delete
- `\e` - escape
- `\f` - form feed
- `\n` – newline
- `\r` – carriage return
- `\s` – space
- `\t` – tab
- `\v` – vertical tab
- `\0` - null byte
- `\xDD` - represents a single byte in hexadecimal (such as `\x13`)
- `\uDDDD` and `\u{D...}` - represents a Unicode codepoint in hexadecimal (such as `\u{1F600}`)

Untuk tambahan, sebuah tanda petik dua (double quote) di dalam sebuah double-quoted string perlu diescape sebagai `\"` dan secara analog, sebuah single quote di dalam single-quoted char list perlu diescap sebagai `\'`, Meskipun demikian, lebih baik mengubah delimiter seperti terlihat di atas dibanding escape mereka.

Sigil juga mendukung heredocs, yaitu 3 double-quote atau single-quote sebagai separator:

```elixir
~s"""
this is
a heredoc string
"""
```

use case yang umum untuk heredoc sigil adalah ketika menulis dokumentasi. Sebagai contoh, menulis escape character di dokumentasi akan menjadi rawan kesalah karena harus melakukan dua kali escap pada beberapa karakter:

```elixir
@doc """
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\\\"foo\\\"")
    "'foo'"

"""
def convert(...)
```

Dengan menggunakan `~S`, masalah ini dapat dihindari:

```elixir
@doc ~S"""
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\"foo\"")
    "'foo'"

"""
def convert(...)
```

**Calendar sigils**
Elxir menawarkan beberapa sigil untuk berurusan dengan berbagai macam jenis times dan dates.

**Date**
Sebuah [%Date{}](https://hexdocs.pm/elixir/1.16/Date.html) struct berisi field `year`, `month`, `day` dan `calendar`. Dapat dibuat menggunakan `~D` sigil:

```elixir
d = ~D[2019-10-31]
~D[2019-10-31]
d.day
31
```

**Time**
[%Time{}](https://hexdocs.pm/elixir/1.16/Time.html) struct berisi field `hour`, `minute`, `second`, `microsecond`, dan `calendar` yang dapat dibuat menggunakan `~T` sigil:

```elixir
t = ~T[23:00:07.0]
~T[23:00:07.0]
t.second
7
```

**NaiveDateTime**
[%NaiveDateTime{}](https://hexdocs.pm/elixir/1.16/NaiveDateTime.html) struct berisi field `Date` dan `Time` yang dapat dibuat menggunakan `~N` sigil:

```elixir
ndt = ~N[2019-10-31 23:00:07]
~N[2019-10-31 23:00:07]
```

Dipanggil naive (naif) karena tidak ada timezonenya. Oleh karena itu, datetime yang diberikan mungkin tidak ada sama sekali atau mungkin ada 2 kali timezone tertentu. misalnya ketika kita memindahkan jam ke belakang dan ke depan untuk waktu musim panas.

**UTCDateTime**
[%DateTime{}](https://hexdocs.pm/elixir/1.16/DateTime.html) struct berisi field yang sama dengan `NaiveDateTime` dengan tambahan field untuk menjejak timezone. `~U` sigil mengijinkan developer untuk membuat sebuah DateTime di UTC timezone.

```elixir
dt = ~U[2019-10-31 19:59:03Z]
~U[2019-10-31 19:59:03Z]
%DateTime{minute: minute, time_zone: time_zone} = dt
~U[2019-10-31 19:59:03Z]
minute
59
time_zone
"Etc/UTC"
```

**Custom sigils**
Seperti telah diisyaratkan di depan bab ini, sigils di elixir extensible. Pada faktanya, menggunakan sigil `~r/foo/i` adalah sama dengan memanggil `sigil_r` dengan sebuah binary dan sebuah char list sebagai argument:

```elixir
sigil_r(<<"foo">>, [?i])
~r"foo"i
```

Kita dapat mengakses dokumentasi `~r` sigil via `sigil_r`:

```elixir
h sigil_r
```

Kita dapat juga menyediakan sigil kita sendiri dengan mengimplementasikan function yang mengikuti `sigil_{character}` pattern. Sebagai contoh, mari implementasikan `~i` sigil yang mengembalikan integer (dengan optional `n` modifier untuk membuatnya negatif):

```elixir
defmodule MySigils do
  def sigil_i(string, []), do: String.to_integer(string)
  def sigil_i(string, [?n]), do: -String.to_integer(string)
end
import MySigils
~i(13)
13
~i(42)n
-42
```

Custom sigil mungkin sebuah lowercase single character atau beberapa uppercase character.

Sigil dapat juga digunakan untuk melakukan compile-time work dengan bantuan macro. Contohnya, regular expression di Elixir dicompile menjadi sebuah representasi efisien selama kompilasi dari source code, mmeskipun begitu melewatkan (skipping) langkah ini di runtime. Jika kamu tertarik di bahasan ini, kamu dapat belajar lebih lanjut tentang macro bagaimana sigil diimplementasi di `Kernel` modul (di mana `sigil_*` function didefinisikan)
