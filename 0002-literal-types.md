# Pelajaran 2 · TypeScript untuk TanStack

## Literal Type: Perbezaan `const` dan `let`

**Misi:** Fahami asas literal type sebelum kembali kepada TanStack Router.

Dalam pelajaran pertama, kita melihat bahawa TanStack Router boleh mengetahui bahawa:

```ts
createFileRoute('/')
```

merujuk tepat kepada route `'/'`.

Tetapi sebelum memahami generic, `FileRoutesByPath` dan type inference, kita perlu memahami satu asas penting terlebih dahulu:

> TypeScript boleh membezakan antara nilai tepat `'/'` dengan type umum `string`.

---

## 1. Fokus pada dua pemboleh ubah mudah

Lupakan TanStack Router buat sementara waktu.

Perhatikan kod ini:

```ts
const home = '/'
let path = '/'
```

Walaupun kedua-duanya mempunyai nilai yang sama, TypeScript boleh memberi type yang berbeza.

```ts
const home = '/'
// type: '/'

let path = '/'
// type: string
```

Kenapa?

---

## 2. `const` menyimpan type yang lebih tepat

Apabila anda menulis:

```ts
const home = '/'
```

nilai `home` tidak boleh ditukar selepas itu.

Kod berikut tidak dibenarkan:

```ts
const home = '/'

home = '/dashboard'
// Error
```

Disebabkan nilainya tidak boleh berubah, TypeScript boleh menyimpan maklumat yang sangat tepat.

Type untuk `home` ialah:

```ts
'/'
```

Bukan sekadar:

```ts
string
```

Type `'/'` bermaksud:

> Pemboleh ubah ini hanya boleh mempunyai satu nilai, iaitu `'/'`.

Ini dipanggil **literal type**.

---

## 3. `let` biasanya menghasilkan type yang lebih umum

Sekarang lihat:

```ts
let path = '/'
```

Pemboleh ubah `path` boleh ditukar kemudian:

```ts
let path = '/'

path = '/dashboard'
path = '/users'
path = '/settings'
```

Oleh sebab nilainya boleh berubah kepada string lain, TypeScript tidak boleh menganggap type tersebut hanya `'/'`.

Jadi TypeScript melebarkan type itu menjadi:

```ts
string
```

Proses ini dipanggil **type widening**.

Contohnya:

```ts
let path = '/'
// type: string
```

TypeScript memahami bahawa `path` mungkin mengandungi mana-mana string pada masa akan datang.

---

## 4. Apakah literal type?

Literal type ialah type yang mewakili satu nilai tepat.

Contoh string literal type:

```ts
type HomePath = '/'
```

Type `HomePath` hanya menerima:

```ts
const path: HomePath = '/'
```

Ini tidak dibenarkan:

```ts
const path: HomePath = '/dashboard'
// Error
```

Contoh literal type lain:

```ts
type Status = 'active'
type Answer = true
type Quantity = 42
```

Setiap type tersebut hanya membenarkan satu nilai:

```ts
const status: Status = 'active'
const answer: Answer = true
const quantity: Quantity = 42
```

Jadi nilai berikut semuanya boleh menjadi literal type:

```ts
'/'
'home'
42
true
```

Manakala berikut ialah type umum:

```ts
string
number
boolean
```

Perbezaannya:

```ts
string
```

boleh mewakili banyak nilai:

```ts
'home'
'users'
'dashboard'
'apa-apa-string'
```

Tetapi literal type:

```ts
'home'
```

hanya mewakili satu nilai:

```ts
'home'
```

---

## 5. Literal type ialah type, bukan sekadar nilai

Apabila kita melihat:

```ts
const home = '/'
```

`'/'` ialah nilai semasa runtime.

Tetapi TypeScript juga boleh menggunakan bentuk yang sama sebagai type:

```ts
let route: '/' = '/'
```

Bahagian kiri:

```ts
route: '/'
```

menggunakan `'/'` sebagai type.

Bahagian kanan:

```ts
= '/'
```

menggunakan `'/'` sebagai nilai.

Contoh lain:

```ts
let page: 'home' = 'home'
```

Di sini:

* `'home'` selepas `:` ialah type.
* `'home'` selepas `=` ialah nilai.

TypeScript menggunakan literal type untuk menggambarkan nilai yang sangat tepat.

---

## 6. Perbezaan `const` dan `let`

Bandingkan kedua-dua contoh berikut.

### Menggunakan `const`

```ts
const page = 'home'
```

Type yang diinfer:

```ts
'home'
```

Ini kerana `page` tidak boleh diberi nilai baru.

### Menggunakan `let`

```ts
let page = 'home'
```

Type yang diinfer:

```ts
string
```

Ini kerana `page` boleh berubah:

```ts
page = 'dashboard'
page = 'settings'
```

Ringkasnya:

```text
const → TypeScript kekalkan type yang lebih tepat
let   → TypeScript biasanya lebarkan kepada type umum
```

---

## 7. Kenapa ini penting dalam TanStack Router?

Dalam:

```ts
src/routes/index.tsx
```

anda mempunyai:

```ts
export const Route = createFileRoute('/')({
  component: Home,
})
```

Argument yang dihantar ialah:

```ts
'/'
```

TypeScript tidak melihatnya sebagai sebarang string.

Ia melihatnya sebagai literal type:

```ts
'/'
```

Jadi TanStack Router boleh mengetahui bahawa route tersebut ialah route home secara tepat.

Secara konsep, TypeScript memahami:

```ts
TFilePath = '/'
```

Bukannya:

```ts
TFilePath = string
```

Maklumat tepat ini membolehkan TanStack Router membezakan antara path yang sah dan path yang tidak sah.

Contohnya:

```tsx
<Link to="/" />
```

boleh diterima kerana `'/'` ialah route yang wujud.

Tetapi:

```tsx
<Link to="/benda-lain" />
```

akan menghasilkan ralat sekiranya route tersebut tidak terdapat dalam route tree.

---

## 8. Apa akan berlaku jika path menjadi `string`?

Perhatikan contoh ini:

```ts
let route = '/'
```

Type `route` ialah:

```ts
string
```

Sekarang cuba:

```ts
createFileRoute(route)
```

TanStack Router mungkin tidak dapat menggunakan `route` sebagai typed file path kerana type tersebut terlalu luas.

`string` boleh mengandungi apa sahaja:

```ts
'/'
'/users'
'/tidak-wujud'
'hello'
'abc'
```

Tetapi `createFileRoute` mahukan salah satu path tepat yang telah dijana dalam `FileRoutesByPath`.

Contohnya:

```ts
'/' | '/users' | '/settings'
```

Type `string` terlalu umum untuk dijamin sebagai salah satu daripada path tersebut.

---

## 9. Literal value terus dalam function call

Anda mungkin tertanya:

> Dalam `createFileRoute('/')`, kita tidak menggunakan `const`. Kenapa `'/'` masih literal type?

Kerana literal tersebut dihantar terus kepada generic function.

```ts
createFileRoute('/')
```

TypeScript boleh menggunakan nilai tepat `'/'` untuk membuat inferens generic type parameter.

Secara konsep:

```ts
createFileRoute<'/'>('/')
```

TypeScript melakukan ini secara automatik melalui type argument inference.

Jadi dua bentuk berikut boleh mengekalkan literal type:

```ts
const route = '/'
createFileRoute(route)
```

dan:

```ts
createFileRoute('/')
```

Tetapi ini mungkin terlalu luas:

```ts
let route = '/'
createFileRoute(route)
```

kerana `route` telah menjadi `string`.

---

## 10. Literal type dengan anotasi

Anda boleh memaksa pemboleh ubah `let` mempunyai literal type tertentu:

```ts
let route: '/' = '/'
```

Sekarang type `route` ialah:

```ts
'/'
```

Walaupun ia menggunakan `let`, ia hanya boleh diberi nilai `'/'`.

Ini dibenarkan:

```ts
route = '/'
```

Tetapi ini tidak dibenarkan:

```ts
route = '/users'
// Error
```

Ini menunjukkan bahawa `let` tidak semestinya sentiasa menjadi `string`.

Perkara yang berlaku ialah:

```ts
let route = '/'
```

tanpa anotasi akan melalui type widening.

Tetapi:

```ts
let route: '/' = '/'
```

mempunyai literal type yang dinyatakan secara jelas.

---

## 11. Menggunakan `as const`

Satu lagi cara untuk mengekalkan literal type ialah menggunakan:

```ts
as const
```

Contohnya:

```ts
let route = '/' as const
```

Type `route` menjadi:

```ts
'/'
```

Walaupun pemboleh ubah menggunakan `let`, TypeScript mengekalkan literal type tersebut.

Namun, oleh sebab typenya hanya `'/'`, ia tidak boleh ditukar kepada string lain:

```ts
route = '/dashboard'
// Error
```

`as const` juga sangat berguna untuk object dan array.

Contohnya:

```ts
const route = {
  path: '/',
} as const
```

Tanpa `as const`, type `path` biasanya ialah:

```ts
string
```

Dengan `as const`, type `path` menjadi:

```ts
'/'
```

---

## 12. Contoh mudah menggunakan function

Bayangkan function berikut:

```ts
function openPage(page: 'home' | 'settings') {
  console.log(page)
}
```

Function ini hanya menerima dua literal value:

```ts
'home'
```

atau:

```ts
'settings'
```

Kod berikut sah:

```ts
openPage('home')
openPage('settings')
```

Kod berikut tidak sah:

```ts
openPage('profile')
// Error
```

Sekarang lihat perbezaan `const` dan `let`.

### Dengan `const`

```ts
const page = 'home'

openPage(page)
```

Ini sah kerana type `page` ialah:

```ts
'home'
```

### Dengan `let`

```ts
let page = 'home'

openPage(page)
```

Ini boleh menghasilkan ralat kerana type `page` ialah:

```ts
string
```

TypeScript tidak boleh menjamin bahawa `page` akan kekal `'home'`.

Ia mungkin berubah:

```ts
page = 'apa-apa'
```

Situasi ini sama dengan bagaimana TanStack Router memeriksa route path.

---

## 13. Cuba dalam editor

Buka:

```text
src/routes/index.tsx
```

Tambahkan sementara:

```ts
const route = '/'
```

Hover pada `route`.

VS Code sepatutnya menunjukkan sesuatu seperti:

```ts
const route: "/"
```

Kemudian tukar kepada:

```ts
let route = '/'
```

Hover sekali lagi.

VS Code sepatutnya menunjukkan:

```ts
let route: string
```

Perubahan kecil daripada `const` kepada `let` menyebabkan TypeScript kehilangan maklumat bahawa nilainya tepat `'/'`.

Selepas selesai, padam kod ujian tersebut.

---

## 14. Cuba ujian tambahan

Tambahkan kod ini:

```ts
const route = '/'
const exactRoute: '/' = route
```

Kod tersebut sepatutnya diterima kerana type `route` ialah `'/'`.

Sekarang cuba:

```ts
let route = '/'
const exactRoute: '/' = route
```

TypeScript akan menghasilkan ralat kerana `route` ialah `string`.

Secara konsep, mesej ralatnya bermaksud:

```text
Type string tidak boleh digunakan sebagai type '/'
```

Ini kerana `string` terlalu luas.

---

## 15. Kuiz ringkas

### Soalan 1

Apakah type bagi kod berikut?

```ts
const a = 'home'
```

Pilihan:

* `'home'`
* `string`
* `any`
* `string[]`

Jawapan:

```ts
'home'
```

Disebabkan `a` ialah `const`, TypeScript mengekalkan literal type yang tepat.

---

### Soalan 2

Apakah type bagi kod berikut?

```ts
let b = 'home'
```

Pilihan:

* `'home'`
* `string`
* `any`
* `string[]`

Jawapan:

```ts
string
```

Disebabkan `b` boleh ditukar, TypeScript melakukan type widening.

---

### Soalan 3

Antara berikut, yang manakah literal type?

* `string`
* `number`
* `boolean`
* `42`

Jawapan:

```ts
42
```

`42` ialah literal type yang hanya mewakili nilai nombor `42`.

`number` pula ialah type umum yang boleh mewakili banyak nombor.

---

### Soalan 4

Kenapa TypeScript boleh membezakan:

```tsx
<Link to="/" />
```

dengan:

```tsx
<Link to="/benda-lain" />
```

Pilihan:

* Sebab panjang string
* Sebab pemeriksaan runtime
* Sebab literal type `'/'`
* Sebab `any`

Jawapan:

```text
Sebab literal type '/'
```

TanStack Router menggunakan literal type untuk membandingkan path dengan route map yang dijana.

---

## 16. Perkara utama yang perlu diingati

### `const` biasanya mengekalkan literal type

```ts
const home = '/'
// type: '/'
```

### `let` biasanya melakukan widening

```ts
let path = '/'
// type: string
```

### Literal type mewakili satu nilai tepat

```ts
'/'
'home'
42
true
```

### Type umum mewakili banyak nilai

```ts
string
number
boolean
```

### TanStack Router memerlukan maklumat yang tepat

```ts
createFileRoute('/')
```

TypeScript menangkap `'/'` sebagai literal type, bukan `string`.

Maklumat inilah yang menjadi asas kepada type-safe routing.

---

# Ringkasan

Perbezaan paling penting ialah:

```ts
const home = '/'
// type: '/'
```

berbanding:

```ts
let path = '/'
// type: string
```

`const` membolehkan TypeScript menyimpan type yang sangat tepat kerana nilainya tidak boleh berubah.

`let` menyebabkan TypeScript melebarkan type kepada `string` kerana nilainya boleh ditukar kemudian.

Dalam TanStack Router, literal type seperti `'/'` membolehkan TypeScript mengetahui route tepat yang sedang digunakan.

Formula ringkasnya:

```text
Nilai tepat
→ literal type
→ route path yang tepat
→ navigation yang type-safe
```

Pelajaran seterusnya ialah **Jenis dan anotasi**, iaitu cara membaca dan menulis perbendaharaan kata type dalam TypeScript.
