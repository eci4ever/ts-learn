# Pelajaran 4 · TypeScript untuk TanStack

## Membaca Ralat TypeScript

**Misi:** Belajar melihat setiap ralat TypeScript sebagai maklumat yang boleh dibaca, bukan mesej misteri yang perlu diteka.

Apabila TypeScript menunjukkan ralat, ia sebenarnya sedang menerangkan tiga perkara:

1. Di mana masalah berlaku.
2. Jenis apa yang diterima.
3. Jenis apa yang sepatutnya digunakan.

Kemahiran utama bukan menghafal semua kod ralat. Kemahiran utama ialah belajar memecahkan mesej itu kepada bahagian kecil.

---

# 1. Anatomi ralat TypeScript

Contoh ralat:

```text
src/demo-errors.ts(1,7): error TS2322: Type 'string' is not assignable to type 'number'.
```

Ralat ini boleh dipecahkan seperti berikut:

```text
src/demo-errors.ts (1,7) : error TS2322 : Type 'string' is not assignable to type 'number'.
│                  │  │          │         │
│                  │  │          │         └─ Penerangan masalah
│                  │  │          └─ Kod ralat
│                  │  └─ Kolum
│                  └─ Baris
└─ Nama fail
```

Tiga bahagian utama yang perlu dibaca ialah:

## Lokasi

```text
src/demo-errors.ts(1,7)
```

Maksudnya:

* Fail: `src/demo-errors.ts`
* Baris: `1`
* Kolum: `7`

Lokasi ini memberitahu anda tempat pertama yang perlu diperiksa.

---

## Kod ralat

```text
TS2322
```

Ini ialah nombor pengenalan bagi jenis ralat tersebut.

Kod ini berguna apabila:

* anda mahu mencari dokumentasi,
* mencari isu yang sama,
* membandingkan ralat dalam repository,
* atau meminta bantuan daripada agent.

Namun, anda tidak perlu menghafal semua kod ralat.

Lebih penting untuk memahami mesej selepasnya.

---

## Mesej ralat

```text
Type 'string' is not assignable to type 'number'.
```

Maksudnya:

> Nilai yang mempunyai type `string` sedang digunakan di tempat yang memerlukan type `number`.

Contohnya:

```ts
const count: number = 'sepuluh'
```

Type sebelah kiri menetapkan:

```ts
number
```

Tetapi nilai sebelah kanan ialah:

```ts
string
```

Kedua-duanya tidak sepadan.

---

# 2. Maksud “assignable”

Perkataan yang sangat kerap muncul dalam ralat TypeScript ialah:

```text
assignable
```

Dalam Bahasa Melayu, ia boleh difahami sebagai:

> Boleh diberikan kepada sesuatu type.

Contohnya:

```ts
const count: number = 10
```

Nilai `10` boleh diberikan kepada pemboleh ubah bertype `number`.

Jadi `number` boleh digunakan di tempat yang memerlukan `number`.

Tetapi:

```ts
const count: number = '10'
```

Walaupun teks `'10'` kelihatan seperti nombor, typenya masih `string`.

Jadi `string` tidak boleh diberikan kepada `number`.

---

# 3. Cara paling mudah membaca mesej

Untuk mesej berikut:

```text
Type 'string' is not assignable to type 'number'.
```

Baca seperti ini:

```text
Type sebenar: string
Type yang diperlukan: number
```

Atau:

> Saya sedang memberikan `string` kepada tempat yang memerlukan `number`.

Gunakan formula ini:

```text
Type yang diberi
→ tidak sesuai dengan
→ type yang diminta
```

---

# 4. Bentuk A: `Type ... is not assignable`

Contoh:

```text
error TS2322: Type 'string' is not assignable to type 'number'.
```

Ini biasanya berlaku semasa assignment.

Contohnya:

```ts
const count: number = 'bukan nombor'
```

Bahagian yang diberi:

```ts
'bukan nombor'
```

mempunyai type:

```ts
string
```

Bahagian yang diperlukan:

```ts
count: number
```

memerlukan:

```ts
number
```

## Cara membaiki

Terdapat dua kemungkinan.

### Betulkan nilainya

```ts
const count: number = 10
```

### Atau betulkan typenya

```ts
const count: string = 'bukan nombor'
```

Pilihan yang betul bergantung pada tujuan sebenar pemboleh ubah itu.

Jangan terus menukar type kepada `any` hanya untuk menghilangkan ralat.

---

# 5. Bentuk B: sifat wajib hilang

Contoh:

```text
error TS2741: Property 'umur' is missing in type '{ nama: string; }'
but required in type 'User'.
```

Bayangkan type berikut:

```ts
type User = {
  nama: string
  umur: number
}
```

Kemudian anda menulis:

```ts
const orang: User = {
  nama: 'Ali',
}
```

TypeScript membandingkan bentuk object yang diberi dengan bentuk `User`.

Object yang diberi mempunyai:

```ts
{
  nama: string
}
```

Tetapi `User` memerlukan:

```ts
{
  nama: string
  umur: number
}
```

Sifat `umur` tiada.

---

## Cara membaca mesej

```text
Property 'umur' is missing
```

bermaksud:

> Object ini tidak mempunyai sifat `umur`.

Bahagian:

```text
but required in type 'User'
```

bermaksud:

> Tetapi type `User` mewajibkan sifat itu.

---

## Cara membaiki

Tambahkan sifat yang hilang:

```ts
const orang: User = {
  nama: 'Ali',
  umur: 25,
}
```

Jika `umur` sebenarnya tidak wajib, jadikannya optional:

```ts
type User = {
  nama: string
  umur?: number
}
```

Tanda:

```ts
?
```

bermaksud sifat itu tidak diwajibkan.

---

# 6. Structural typing

Ralat sifat hilang berkait rapat dengan konsep **structural typing**.

TypeScript tidak hanya melihat nama type.

TypeScript melihat bentuk data.

Contohnya:

```ts
type User = {
  nama: string
  umur: number
}
```

Object berikut boleh diterima walaupun tidak ditulis menggunakan constructor khas:

```ts
const pengguna = {
  nama: 'Ali',
  umur: 25,
}

const orang: User = pengguna
```

Ini sah kerana bentuk `pengguna` sepadan dengan bentuk `User`.

TypeScript bertanya:

```text
Adakah semua sifat wajib wujud?
Adakah type setiap sifat sepadan?
```

Jika ya, assignment diterima.

---

# 7. Bentuk C: argument tidak sesuai dengan parameter

Contoh:

```text
error TS2345: Argument of type 'string'
is not assignable to parameter of type 'number'.
```

Bayangkan function berikut:

```ts
function gandakan(nilai: number) {
  return nilai * 2
}
```

Kemudian function dipanggil dengan:

```ts
gandakan('empat')
```

Argument yang dihantar ialah:

```ts
'empat'
```

Typenya:

```ts
string
```

Tetapi parameter function memerlukan:

```ts
number
```

---

## Bezakan argument dan parameter

Dalam function:

```ts
function gandakan(nilai: number) {
  return nilai * 2
}
```

`nilai` ialah parameter.

Dalam panggilan:

```ts
gandakan(4)
```

`4` ialah argument.

Jadi mesej:

```text
Argument of type 'string'
is not assignable to parameter of type 'number'
```

bermaksud:

> Nilai yang dihantar ke function salah type.

---

## Cara membaiki

Hantar argument yang betul:

```ts
gandakan(4)
```

Atau ubah function jika memang mahu menerima string:

```ts
function gandakan(nilai: string) {
  return nilai.repeat(2)
}
```

Namun perubahan function mesti sepadan dengan tujuan sebenar kod.

---

# 8. Ralat pada array

Contoh:

```ts
const nilai = [1, 2, 3]

nilai.push('empat')
```

TypeScript mengesan `nilai` sebagai:

```ts
number[]
```

Method `push` untuk array ini menerima:

```ts
number
```

Tetapi anda menghantar:

```ts
string
```

Maka ralatnya lebih kurang:

```text
Argument of type 'string'
is not assignable to parameter of type 'number'.
```

Cara membaiki:

```ts
nilai.push(4)
```

Jika array memang perlu menerima nombor dan string:

```ts
const nilai: Array<number | string> = [1, 2, 3]

nilai.push('empat')
```

Tetapi jangan jadikan union tanpa sebab. Pastikan struktur data itu memang memerlukannya.

---

# 9. Ralat generic

Perhatikan function generic berikut:

```ts
function gema<T>(nilai: T): T {
  return nilai
}
```

Kemudian:

```ts
const hasil: string = gema(42)
```

Apabila `gema(42)` dipanggil, TypeScript menyimpulkan:

```ts
T = number
```

Jadi return type function ialah:

```ts
number
```

Secara konsep:

```ts
gema<number>(42)
```

Tetapi pemboleh ubah `hasil` memerlukan:

```ts
string
```

Maka ralatnya:

```text
Type 'number' is not assignable to type 'string'.
```

---

## Cara membaca ralat generic

Jangan terus fokus kepada huruf `T`.

Cari tiga perkara:

```text
Apakah argument yang dihantar?
T disimpulkan sebagai type apa?
Function memulangkan type apa?
```

Untuk contoh tadi:

```text
Argument: 42
T: number
Return type: number
Destination type: string
```

Jadi masalahnya jelas:

```text
number tidak boleh diberikan kepada string
```

Cara membaiki:

```ts
const hasil: number = gema(42)
```

atau:

```ts
const hasil: string = gema('42')
```

---

# 10. Mesej union literal

Dalam TanStack Router, anda mungkin melihat ralat seperti:

```text
Argument of type 'string' is not assignable to parameter of type
'"/" | "/about"'.
```

Bahagian:

```ts
'/' | '/about'
```

ialah union literal type.

Maksudnya hanya dua nilai dibenarkan:

```ts
'/'
```

atau:

```ts
'/about'
```

Tetapi nilai yang dihantar mempunyai type umum:

```ts
string
```

`string` terlalu luas kerana ia mungkin mengandungi:

```ts
'/'
'/about'
'/users'
'/tidak-wujud'
'hello'
```

TypeScript tidak boleh menjamin nilai itu ialah route yang sah.

---

# 11. Contoh TanStack: `Link to={someString}`

Contoh:

```tsx
let someString = '/'

<Link to={someString}>Home</Link>
```

Oleh sebab `someString` menggunakan `let`, typenya biasanya:

```ts
string
```

Tetapi prop `to` mungkin hanya menerima:

```ts
'/' | '/about'
```

Jadi TypeScript menghasilkan ralat.

---

## Penyelesaian pertama: gunakan literal

```tsx
<Link to="/">Home</Link>
```

---

## Penyelesaian kedua: gunakan `const`

```tsx
const homePath = '/'

<Link to={homePath}>Home</Link>
```

Type `homePath` ialah:

```ts
'/'
```

---

## Penyelesaian ketiga: beri type route yang sah

```ts
const path: '/' | '/about' = '/'
```

Kemudian:

```tsx
<Link to={path}>Home</Link>
```

---

## Penyelesaian yang perlu dielakkan

```tsx
<Link to={someString as any}>Home</Link>
```

Ini hanya menutup pemeriksaan TypeScript.

Ia tidak membuktikan bahawa route itu sah.

---

# 12. Membaca ralat panjang

Kadangkala ralat TypeScript sangat panjang:

```text
Type 'SomeComplexGenericType<...>' is not assignable to type
'AnotherComplexGenericType<...>'.
```

Jangan cuba memahami seluruh mesej sekaligus.

Gunakan kaedah berikut.

## Langkah 1: Cari ayat utama

Cari frasa seperti:

```text
is not assignable to
is missing
does not exist
cannot be used
expected
```

---

## Langkah 2: Cari type sebenar

Biasanya muncul di sebelah kiri:

```text
Type 'string'
```

atau:

```text
Argument of type 'string'
```

---

## Langkah 3: Cari type yang diperlukan

Biasanya muncul di hujung:

```text
type 'number'
parameter of type 'number'
type '"/" | "/about"'
```

---

## Langkah 4: Cari property pertama yang bermasalah

Dalam ralat object yang panjang, TypeScript mungkin menerangkan beberapa lapisan.

Contohnya:

```text
Type '{ user: { id: string } }' is not assignable to type 'Session'.
Property 'role' is missing in type '{ id: string }'
but required in type 'User'.
```

Masalah sebenar ialah:

```text
Property 'role' is missing
```

Mulakan dari bahagian paling bawah atau paling spesifik.

---

# 13. Ralat editor berbanding terminal

## Dalam VS Code

Ralat biasanya ditunjukkan dengan garis merah.

Hover pada garis tersebut untuk melihat mesej.

Anda juga boleh menekan:

```text
Cmd + .
```

pada macOS, atau:

```text
Ctrl + .
```

pada Windows dan Linux.

Ini membuka menu Quick Fix.

Contoh cadangan:

* tambah import,
* tambah property,
* tukar type,
* buang import tidak digunakan,
* tambah `async`,
* tambah return type.

Quick Fix boleh membantu, tetapi jangan terima cadangan tanpa memahami perubahan yang dibuat.

---

## Dalam terminal

Jalankan:

```bash
npx tsc --noEmit
```

`tsc` ialah TypeScript compiler.

Flag:

```text
--noEmit
```

bermaksud:

> Periksa type sahaja, jangan hasilkan fail JavaScript.

Ini berguna untuk memastikan seluruh project bebas daripada ralat type.

---

# 14. Latihan dalam `src/demo-errors.ts`

Anggap fail berikut mempunyai empat ralat:

```ts
const count: number = 'bukan nombor'

type User = {
  nama: string
  umur: number
}

const orang: User = {
  nama: 'Ali',
}

function gema<T>(nilai: T): T {
  return nilai
}

const hasil: string = gema(42)

const nilai = [1, 2, 3]
nilai.push('empat')
```

---

# 15. Ralat pertama

Kod:

```ts
const count: number = 'bukan nombor'
```

Type sebenar:

```ts
string
```

Type diperlukan:

```ts
number
```

Pembaikan:

```ts
const count: number = 10
```

---

# 16. Ralat kedua

Kod:

```ts
const orang: User = {
  nama: 'Ali',
}
```

Sifat yang hilang:

```ts
umur
```

Pembaikan:

```ts
const orang: User = {
  nama: 'Ali',
  umur: 25,
}
```

---

# 17. Ralat ketiga

Kod:

```ts
const hasil: string = gema(42)
```

Argument:

```ts
42
```

Menyebabkan:

```ts
T = number
```

Return type:

```ts
number
```

Tetapi destination type ialah:

```ts
string
```

Pembaikan:

```ts
const hasil: number = gema(42)
```

---

# 18. Ralat keempat

Kod:

```ts
const nilai = [1, 2, 3]
nilai.push('empat')
```

Type array:

```ts
number[]
```

Type yang dihantar ke `push`:

```ts
string
```

Pembaikan:

```ts
nilai.push(4)
```

---

# 19. Fail yang telah dibetulkan

```ts
const count: number = 10

type User = {
  nama: string
  umur: number
}

const orang: User = {
  nama: 'Ali',
  umur: 25,
}

function gema<T>(nilai: T): T {
  return nilai
}

const hasil: number = gema(42)

const nilai = [1, 2, 3]
nilai.push(4)
```

Selepas membetulkan semua ralat, jalankan:

```bash
npx tsc --noEmit
```

Jika tiada output dan command selesai dengan berjaya, ini biasanya bermaksud tiada ralat TypeScript ditemui.

Selepas latihan selesai, padam:

```text
src/demo-errors.ts
```

jika fail itu hanya digunakan untuk latihan.

---

# 20. Strategi membaca ralat sebenar

Apabila menemui ralat, gunakan urutan berikut.

## Langkah 1: Buka lokasi

Contoh:

```text
src/demo-errors.ts(17,12)
```

Pergi ke fail, baris dan kolum tersebut.

---

## Langkah 2: Kenal pasti operasi

Tanya:

```text
Adakah ini assignment?
Panggilan function?
Return value?
Object?
Array?
Prop component?
```

---

## Langkah 3: Cari type sebenar

Contoh:

```text
Type 'string'
Argument of type 'string'
```

---

## Langkah 4: Cari type yang diperlukan

Contoh:

```text
type 'number'
parameter of type 'number'
type '"/" | "/about"'
```

---

## Langkah 5: Bandingkan kedua-duanya

Contoh:

```text
Ada: string
Perlu: number
```

atau:

```text
Ada: string
Perlu: '/' | '/about'
```

---

## Langkah 6: Betulkan sumber masalah

Pembaikan mungkin melibatkan:

* menukar nilai,
* menukar anotasi,
* menambah property,
* membetulkan parameter,
* membetulkan return type,
* menggunakan literal type,
* membetulkan generic,
* atau membetulkan data daripada API.

Elakkan sekadar memaksa type dengan `as any`.

---

# 21. Kesalahan biasa ketika membaiki ralat

## Menggunakan `any`

Contoh:

```ts
const count: number = 'bukan nombor' as any
```

Ralat hilang, tetapi data masih salah.

TypeScript tidak lagi melindungi kod tersebut.

---

## Menggunakan type assertion tanpa bukti

Contoh:

```ts
const path = input as '/'
```

Ini memberitahu TypeScript bahawa `input` pasti `'/'`, tetapi tiada pemeriksaan runtime dilakukan.

Jika `input` sebenarnya `'/benda-lain'`, aplikasi masih boleh bermasalah.

---

## Mengubah type supaya menerima semua benda

Contoh:

```ts
const count: string | number | boolean | object = 'bukan nombor'
```

Ini mungkin menghilangkan ralat, tetapi menjadikan kod terlalu umum dan sukar digunakan.

Type sepatutnya menerangkan data sebenar, bukan sekadar memuaskan compiler.

---

## Hanya melihat baris merah

Kadangkala baris yang ditunjukkan bukan punca asal.

Contohnya, data datang daripada function:

```ts
const user: User = getUser()
```

Ralat muncul pada assignment, tetapi masalah mungkin berada dalam return type `getUser()`.

Ikuti type ke sumbernya.

---

# 22. Kuiz

## Soalan 1

Dalam:

```text
error TS2322
```

apakah `2322`?

Pilihan:

* Baris ralat
* Kod ralat
* Fail ralat
* Kolum ralat

Jawapan:

```text
Kod ralat
```

---

## Soalan 2

Apakah maksud:

```text
Type 'string' is not assignable to type 'number'
```

Pilihan:

* Nilai dan type tidak sepadan
* String ditukar menjadi nombor
* Number ditukar menjadi string
* Ralat runtime

Jawapan:

```text
Nilai dan type tidak sepadan
```

Lebih tepat, nilai `string` digunakan di tempat yang memerlukan `number`.

---

## Soalan 3

Apakah maksud:

```text
Property 'umur' is missing
```

Pilihan:

* Ralat runtime
* Import hilang
* Sifat wajib tiada
* Type kosong

Jawapan:

```text
Sifat wajib tiada
```

---

## Soalan 4

Kenapa kod berikut boleh menghasilkan ralat?

```tsx
<Link to={someString} />
```

Pilihan:

* Sebab runtime
* Sebab CSS
* Sebab `any`
* Path perlu type literal yang sah

Jawapan:

```text
Path perlu type literal yang sah
```

`someString` mungkin bertype `string`, sedangkan router hanya menerima union route seperti:

```ts
'/' | '/about'
```

---

# 23. Glosari ringkas

## Error location

Lokasi fail, baris dan kolum tempat ralat dikesan.

```text
src/file.ts(10,5)
```

---

## Error code

Kod rujukan ralat TypeScript.

```text
TS2322
```

---

## Assignable

Sesuatu type boleh digunakan di tempat yang memerlukan type lain.

---

## Argument

Nilai yang dihantar semasa function dipanggil.

```ts
gandakan(4)
```

`4` ialah argument.

---

## Parameter

Pemboleh ubah yang diisytiharkan dalam function.

```ts
function gandakan(nilai: number) {}
```

`nilai` ialah parameter.

---

## Structural typing

TypeScript membandingkan bentuk object, termasuk sifat dan type setiap sifat.

---

## Type assertion

Arahan kepada TypeScript untuk mempercayai type tertentu.

```ts
value as string
```

Type assertion tidak menukar nilai pada runtime.

---

# 24. Ringkasan utama

Ralat TypeScript biasanya mempunyai tiga bahagian:

```text
Lokasi
+ Kod ralat
+ Mesej
```

Untuk membaca mesej, kenal pasti:

```text
Type yang ada
Type yang diperlukan
Kenapa kedua-duanya tidak sepadan
```

Tiga bentuk ralat paling biasa ialah:

```text
Type tidak boleh diberikan
Property wajib hilang
Argument tidak sesuai dengan parameter
```

Dalam TanStack Router, ralat seperti:

```text
Type 'string' is not assignable to type '"/" | "/about"'
```

bermaksud nilai anda terlalu umum.

Router memerlukan salah satu literal path yang sah.

Formula membaca ralat:

```text
Di mana?
→ Apa yang diberi?
→ Apa yang diperlukan?
→ Apa perbezaannya?
→ Betulkan sumbernya
```

Ralat TypeScript bukan sekadar halangan.

Ia ialah laporan perbandingan type yang memberitahu anda dengan tepat di mana bentuk data atau penggunaan API tidak sepadan.

Pelajaran seterusnya akan menerangkan bagaimana type daripada `loader` mengalir secara automatik ke component melalui `Route.useLoaderData()`.
