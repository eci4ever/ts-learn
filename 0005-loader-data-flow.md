# Pelajaran 5 · TypeScript untuk TanStack

## Bagaimana Type `loader` Mengalir ke Komponen

**Misi:** Memahami dengan tepat bagaimana data daripada `loader` mendapat type secara automatik apabila digunakan dalam komponen.

Salah satu kekuatan utama TanStack Router ialah anda biasanya tidak perlu menulis type secara manual untuk data route.

Anda hanya menulis nilai yang dipulangkan oleh `loader`, kemudian TypeScript membawa type itu ke object `Route` dan akhirnya ke:

```ts
Route.useLoaderData()
```

Keseluruhan alirannya ialah:

```text
Return type loader
        ↓
Type loaderData pada Route
        ↓
Return type useLoaderData()
```

Tiada anotasi manual diperlukan.

TypeScript membuat inferens daripada nilai yang anda tulis.

---

# 1. Rantaian type dalam satu gambar

```text
loader return type
        ↓
Route loaderData type
        ↓
Route.useLoaderData() return type
```

Dalam bentuk penerangan:

```text
Anda menulis data dalam loader
        ↓
TypeScript mengesan bentuk data
        ↓
TanStack Router menyimpan type itu dalam Route
        ↓
useLoaderData() memulangkan type yang sama
```

Contohnya:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

TypeScript mengesan return type loader sebagai:

```ts
{
  name: string
  age: number
}
```

Kemudian dalam komponen:

```ts
const data = Route.useLoaderData()
```

Type `data` juga menjadi:

```ts
{
  name: string
  age: number
}
```

---

# 2. Langkah pertama: loader yang anda tulis

Buka:

```text
src/routes/index.tsx
```

Tambahkan `loader`:

```tsx
export const Route = createFileRoute('/')({
  loader: () => ({
    name: 'Ali',
    age: 30,
  }),
  component: Home,
})

function Home() {
  const data = Route.useLoaderData()

  return <div>{data.name}</div>
}
```

Anda tidak menulis type seperti ini:

```ts
type LoaderData = {
  name: string
  age: number
}
```

Anda juga tidak menulis:

```ts
const data: LoaderData = Route.useLoaderData()
```

TypeScript dapat mengetahui semuanya daripada nilai ini:

```ts
{
  name: 'Ali',
  age: 30,
}
```

---

# 3. Bagaimana TypeScript mengesan return type loader

Loader ialah sebuah function:

```ts
() => ({
  name: 'Ali',
  age: 30,
})
```

TypeScript melihat nilai yang dipulangkan.

Nilai tersebut mempunyai bentuk:

```ts
{
  name: string
  age: number
}
```

Kenapa `name` menjadi `string`, bukan literal type `'Ali'`?

Kerana property object biasanya dianggap boleh mengandungi nilai string lain kemudian.

Secara umum, TypeScript melakukan widening pada property object:

```ts
const user = {
  name: 'Ali',
  age: 30,
}
```

Type `user` biasanya:

```ts
{
  name: string
  age: number
}
```

Jadi return type loader juga diinfer sebagai:

```ts
{
  name: string
  age: number
}
```

---

# 4. Anda boleh melihat inferens secara berasingan

Bayangkan loader itu ditulis sebagai function biasa:

```ts
const loader = () => ({
  name: 'Ali',
  age: 30,
})
```

TypeScript secara konsep mengesan:

```ts
const loader: () => {
  name: string
  age: number
}
```

Jika anda mendapatkan return typenya:

```ts
type LoaderResult = ReturnType<typeof loader>
```

Maka `LoaderResult` ialah:

```ts
{
  name: string
  age: number
}
```

TanStack Router melakukan konsep yang sama, tetapi melalui generic type dalaman yang lebih kompleks.

---

# 5. Route menyimpan type loader

Daripada pelajaran sebelumnya, kita tahu bahawa:

```ts
createFileRoute('/')
```

menghasilkan object route yang mempunyai type khusus untuk route `'/'`.

Selepas anda menambah `loader`, type route itu juga menyimpan maklumat tentang loader tersebut.

Secara konsep, bayangkan type route seperti ini:

```ts
type RouteType = {
  fullPath: '/'
  loaderData: {
    name: string
    age: number
  }
}
```

Type sebenar TanStack Router lebih kompleks, tetapi idea asasnya sama.

Object `Route` membawa dua maklumat penting:

```text
Identiti route
+
Type data loader
```

Dalam contoh ini:

```text
Route path: '/'
Loader data: { name: string; age: number }
```

---

# 6. Type tidak disimpan sebagai data runtime

Penting untuk memahami bahawa TypeScript tidak memasukkan type ke dalam JavaScript semasa runtime.

Apabila kita berkata:

> Route menyimpan type loaderData

kita bermaksud object `Route` mempunyai maklumat itu dalam sistem type TypeScript.

Pada runtime, JavaScript hanya menjalankan function loader dan menerima object biasa.

Pada compile time, TypeScript mengetahui bentuk object tersebut.

Jadi terdapat dua lapisan:

```text
Runtime:
loader dijalankan dan memulangkan data

Compile time:
TypeScript memeriksa bentuk data tersebut
```

---

# 7. Bagaimana TanStack mengambil return type loader

TanStack Router menggunakan type operator untuk mendapatkan return type loader.

Versi ringkasnya seperti ini:

```ts
type LooseAsyncReturnType<T> =
  T extends (...args: any[]) => infer TReturn
    ? TReturn extends Promise<infer TResolved>
      ? TResolved
      : TReturn
    : never
```

Kod ini kelihatan rumit, tetapi tugasnya hanya:

1. Pastikan `T` ialah function.
2. Ambil return type function itu.
3. Jika return type ialah `Promise`, ambil nilai di dalam Promise.
4. Jika bukan Promise, gunakan return type asal.

---

# 8. Memahami `infer`

Perhatikan bahagian ini:

```ts
T extends (...args: any[]) => infer TReturn
```

Maksudnya:

> Jika `T` ialah function, tangkap return typenya dan beri nama `TReturn`.

Contohnya:

```ts
type GetReturn<T> =
  T extends (...args: any[]) => infer R
    ? R
    : never
```

Jika digunakan pada:

```ts
type Result = GetReturn<() => number>
```

TypeScript mendapat:

```ts
type Result = number
```

Kerana function tersebut memulangkan `number`.

Contoh lain:

```ts
type Result = GetReturn<() => string>
```

Hasilnya:

```ts
type Result = string
```

`infer` membolehkan TypeScript mengambil sebahagian type daripada type yang lebih besar.

---

# 9. Memahami conditional type

Struktur ini:

```ts
T extends Something
  ? TypeJikaBenar
  : TypeJikaSalah
```

dipanggil **conditional type**.

Ia serupa dengan ternary expression dalam JavaScript:

```ts
condition ? trueValue : falseValue
```

Tetapi conditional type berlaku dalam sistem type.

Contoh:

```ts
type IsString<T> =
  T extends string
    ? true
    : false
```

Maka:

```ts
type A = IsString<string>
// true

type B = IsString<number>
// false
```

Dalam TanStack Router, conditional type digunakan untuk memeriksa:

```text
Adakah loader memulangkan Promise?
```

---

# 10. Loader synchronous

Contoh loader biasa:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

Return type function ialah:

```ts
{
  name: string
  age: number
}
```

Ia bukan Promise.

Jadi type operator memilih return type asal:

```ts
TReturn
```

Hasil loaderData:

```ts
{
  name: string
  age: number
}
```

---

# 11. Loader asynchronous

Sekarang ubah loader kepada:

```ts
loader: async () => ({
  name: 'Ali',
  age: 30,
})
```

Function `async` sentiasa memulangkan Promise.

Jadi return type function ialah:

```ts
Promise<{
  name: string
  age: number
}>
```

Namun `Route.useLoaderData()` tidak memulangkan Promise.

Ia memulangkan data yang telah diselesaikan:

```ts
{
  name: string
  age: number
}
```

---

# 12. Bagaimana Promise dibuka

Perhatikan bahagian ini:

```ts
TReturn extends Promise<infer TResolved>
  ? TResolved
  : TReturn
```

Jika:

```ts
TReturn =
  Promise<{
    name: string
    age: number
  }>
```

TypeScript memadankan bentuk itu dengan:

```ts
Promise<infer TResolved>
```

Lalu ia menyimpulkan:

```ts
TResolved = {
  name: string
  age: number
}
```

Hasil akhirnya ialah:

```ts
{
  name: string
  age: number
}
```

Promise telah dibuka pada tahap type.

---

# 13. Contoh ringkas membuka Promise

Kita boleh mencipta type sendiri:

```ts
type UnwrapPromise<T> =
  T extends Promise<infer Value>
    ? Value
    : T
```

Contoh:

```ts
type A = UnwrapPromise<Promise<string>>
```

Hasil:

```ts
string
```

Contoh lain:

```ts
type B = UnwrapPromise<number>
```

Hasil:

```ts
number
```

Jika type itu Promise, ambil nilai dalamnya.

Jika bukan Promise, kekalkan type asal.

---

# 14. Kenapa komponen tidak menerima Promise

Dengan loader async:

```ts
loader: async () => ({
  name: 'Ali',
  age: 30,
})
```

Return type loader ialah:

```ts
Promise<{
  name: string
  age: number
}>
```

Tetapi hook:

```ts
const data = Route.useLoaderData()
```

memberikan:

```ts
{
  name: string
  age: number
}
```

Bukan:

```ts
Promise<{
  name: string
  age: number
}>
```

Sebab TanStack Router menunggu loader selesai sebelum menyediakan loader data kepada route component.

Pada tahap type, TanStack membuka Promise dan mengambil resolved value type.

Itulah sebab anda boleh terus menulis:

```tsx
return <div>{data.name}</div>
```

Anda tidak perlu menulis:

```ts
data.then(...)
```

---

# 15. `useLoaderData()` ialah method pada Route

Kod berikut:

```ts
Route.useLoaderData()
```

bukan global hook yang meneka route semasa.

Ia ialah method pada object `Route` tertentu.

Dalam contoh ini:

```ts
export const Route = createFileRoute('/')({
  loader: () => ({
    name: 'Ali',
    age: 30,
  }),
  component: Home,
})
```

Object `Route` mengetahui:

```text
Saya ialah route '/'
Loader saya memulangkan { name: string; age: number }
```

Jadi apabila anda memanggil:

```ts
Route.useLoaderData()
```

hook itu boleh menggunakan type loader milik route tersebut.

---

# 16. Kenapa method pada Route lebih selamat

Bayangkan aplikasi mempunyai dua route.

Route home:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

Route posts:

```ts
loader: () => ({
  posts: ['Post 1', 'Post 2'],
})
```

Setiap object route mempunyai loaderData berbeza.

Untuk home:

```ts
HomeRoute.useLoaderData()
```

Type:

```ts
{
  name: string
  age: number
}
```

Untuk posts:

```ts
PostsRoute.useLoaderData()
```

Type:

```ts
{
  posts: string[]
}
```

Oleh sebab hook datang daripada object route tertentu, TypeScript tidak mencampurkan kedua-dua data tersebut.

---

# 17. Bentuk type hook secara ringkas

Dalam type TanStack Router, idea tersebut lebih kurang:

```ts
type UseLoaderDataRoute<TId> =
  () => RouteById<TId>['types']['loaderData']
```

Baca langkah demi langkah.

Bahagian pertama:

```ts
<TId>
```

ialah ID route.

Bahagian:

```ts
RouteById<TId>
```

bermaksud:

> Cari route berdasarkan ID ini.

Bahagian:

```ts
['types']
```

bermaksud:

> Ambil property type route tersebut.

Bahagian:

```ts
['loaderData']
```

bermaksud:

> Ambil type loaderData milik route itu.

Jadi keseluruhan type bermaksud:

> Function yang memulangkan loaderData untuk route dengan ID `TId`.

---

# 18. Dua inferens dalam satu rantaian

Terdapat dua inferens utama.

## Inferens pertama: identiti route

Daripada:

```ts
createFileRoute('/')
```

TypeScript menyimpulkan:

```ts
TFilePath = '/'
```

Jadi Route mengetahui identitinya.

---

## Inferens kedua: return type loader

Daripada:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

TypeScript menyimpulkan:

```ts
LoaderData = {
  name: string
  age: number
}
```

Kemudian kedua-dua maklumat digabungkan:

```text
Route '/'
mempunyai loaderData
{ name: string; age: number }
```

Hook route itu kemudian memulangkan type tersebut.

---

# 19. Keseluruhan type flow

Kod asal:

```tsx
export const Route = createFileRoute('/')({
  loader: () => ({
    name: 'Ali',
    age: 30,
  }),
  component: Home,
})

function Home() {
  const data = Route.useLoaderData()

  return <div>{data.name}</div>
}
```

Aliran type:

```text
'/'
↓
TFilePath = '/'
↓
Route dikenal pasti sebagai route home
```

Pada masa yang sama:

```text
loader memulangkan object
↓
{ name: string; age: number }
↓
Disimpan sebagai loaderData route
```

Kemudian:

```text
Route.useLoaderData()
↓
Ambil loaderData milik route '/'
↓
{ name: string; age: number }
```

---

# 20. Autocomplete datang daripada type flow

Apabila anda menulis:

```ts
const data = Route.useLoaderData()
```

kemudian menaip:

```ts
data.
```

editor boleh mencadangkan:

```ts
data.name
data.age
```

Ini berlaku kerana editor mengetahui type `data`.

Jika anda menulis:

```ts
data.bogus
```

TypeScript menghasilkan ralat seperti:

```text
Property 'bogus' does not exist on type
'{ name: string; age: number; }'.
```

Ini membuktikan bahawa `data` bukan `any`.

Jika `data` ialah `any`, TypeScript akan menerima sebarang property.

---

# 21. Apa berlaku apabila loader berubah?

Katakan loader asal ialah:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

Komponen boleh menggunakan:

```ts
data.name
data.age
```

Sekarang ubah loader:

```ts
loader: () => ({
  username: 'ali30',
  active: true,
})
```

TypeScript mengemas kini loaderData menjadi:

```ts
{
  username: string
  active: boolean
}
```

Kod lama:

```ts
data.name
```

akan menghasilkan ralat.

Autocomplete baru menjadi:

```ts
data.username
data.active
```

Ini menunjukkan bahawa komponen sentiasa mengikuti bentuk data loader semasa.

---

# 22. Contoh loader dengan array

```tsx
export const Route = createFileRoute('/')({
  loader: () => ({
    users: [
      { id: 1, name: 'Ali' },
      { id: 2, name: 'Siti' },
    ],
  }),
  component: Home,
})
```

TypeScript mengesan loaderData sebagai:

```ts
{
  users: {
    id: number
    name: string
  }[]
}
```

Dalam komponen:

```tsx
function Home() {
  const data = Route.useLoaderData()

  return (
    <ul>
      {data.users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

TypeScript mengetahui bahawa:

```ts
user.id
```

ialah `number`, dan:

```ts
user.name
```

ialah `string`.

Anda tidak perlu menulis type `User` untuk mendapatkan autocomplete asas ini.

---

# 23. Contoh loader async dengan `fetch`

```tsx
export const Route = createFileRoute('/')({
  loader: async () => {
    const response = await fetch('/api/users')
    const users = await response.json()

    return {
      users,
    }
  },
  component: Home,
})
```

Terdapat satu perkara penting di sini.

Dalam banyak konfigurasi TypeScript, return type:

```ts
response.json()
```

boleh menjadi `any`.

Jika `users` ialah `any`, TypeScript tidak boleh mengesan struktur sebenar data API.

Jadi walaupun route type flow berfungsi, ia hanya boleh membawa type yang diterima daripada loader.

Jika loader memulangkan `any`, hook juga mungkin menerima data yang tidak selamat.

---

# 24. Type flow tidak boleh mencipta maklumat yang tiada

TanStack Router membawa type daripada loader ke komponen.

Tetapi ia tidak boleh mengetahui bentuk API secara automatik jika sumber data tidak bertype.

Contohnya:

```ts
const users = await response.json()
```

Jika `users` ialah `any`, loaderData mungkin menjadi:

```ts
{
  users: any
}
```

Kemudian:

```ts
data.users.bogus.whatever
```

mungkin tidak menghasilkan ralat.

Masalahnya bukan pada TanStack Router.

Masalahnya ialah data masuk ke loader sebagai `any`.

---

# 25. Memberi type kepada data API

Anda boleh menentukan type:

```ts
type User = {
  id: number
  name: string
}
```

Kemudian:

```tsx
export const Route = createFileRoute('/')({
  loader: async () => {
    const response = await fetch('/api/users')
    const users: User[] = await response.json()

    return {
      users,
    }
  },
  component: Home,
})
```

Sekarang loaderData ialah:

```ts
{
  users: User[]
}
```

Dan komponen mendapat type itu secara automatik:

```tsx
function Home() {
  const data = Route.useLoaderData()

  return (
    <div>
      {data.users.map((user) => (
        <p key={user.id}>{user.name}</p>
      ))}
    </div>
  )
}
```

Walaupun type diberikan di sempadan API, anda masih tidak perlu menulis type pada `useLoaderData()`.

---

# 26. Validasi runtime masih penting

Type assertion seperti:

```ts
const users = await response.json() as User[]
```

hanya memberitahu TypeScript supaya mempercayai anda.

Ia tidak memeriksa data sebenar daripada server.

Jika server memulangkan:

```json
{
  "message": "error"
}
```

TypeScript tidak dapat mengesannya semasa runtime.

Untuk data luar seperti API, validasi runtime menggunakan schema validator boleh memberikan keselamatan tambahan.

Namun prinsip type flow masih sama:

```text
Data yang telah ditaip atau disahkan
↓
Loader return type
↓
Route loaderData
↓
useLoaderData()
```

---

# 27. Loader yang memulangkan primitive

Loader tidak semestinya memulangkan object.

Contohnya:

```tsx
export const Route = createFileRoute('/')({
  loader: () => 'Selamat datang',
  component: Home,
})
```

Return type loader ialah:

```ts
string
```

Dalam komponen:

```tsx
function Home() {
  const message = Route.useLoaderData()

  return <div>{message}</div>
}
```

Type `message` ialah:

```ts
string
```

---

# 28. Loader yang memulangkan array

```tsx
export const Route = createFileRoute('/')({
  loader: () => ['Ali', 'Siti', 'Abu'],
  component: Home,
})
```

Return type loader ialah:

```ts
string[]
```

Dalam komponen:

```tsx
function Home() {
  const names = Route.useLoaderData()

  return (
    <ul>
      {names.map((name) => (
        <li key={name}>{name}</li>
      ))}
    </ul>
  )
}
```

Type `names` ialah:

```ts
string[]
```

---

# 29. Loader yang boleh memulangkan bentuk berbeza

Contohnya:

```ts
loader: () => {
  if (Math.random() > 0.5) {
    return {
      status: 'success',
      data: 'Hello',
    }
  }

  return {
    status: 'error',
    message: 'Gagal',
  }
}
```

TypeScript akan menghasilkan union type.

Secara konsep:

```ts
{
  status: string
  data: string
  message?: undefined
}
|
{
  status: string
  message: string
  data?: undefined
}
```

Untuk mendapatkan discriminated union yang lebih tepat, anda boleh mengekalkan literal status:

```ts
loader: () => {
  if (Math.random() > 0.5) {
    return {
      status: 'success' as const,
      data: 'Hello',
    }
  }

  return {
    status: 'error' as const,
    message: 'Gagal',
  }
}
```

Kemudian dalam komponen:

```tsx
function Home() {
  const result = Route.useLoaderData()

  if (result.status === 'success') {
    return <div>{result.data}</div>
  }

  return <div>{result.message}</div>
}
```

TypeScript boleh melakukan narrowing berdasarkan `status`.

---

# 30. Loader boleh menggunakan parameter route

Contohnya, route:

```text
/users/$userId
```

mungkin mempunyai loader seperti:

```tsx
export const Route = createFileRoute('/users/$userId')({
  loader: ({ params }) => {
    return {
      id: params.userId,
      name: 'Ali',
    }
  },
  component: UserPage,
})
```

Type loaderData diinfer sebagai:

```ts
{
  id: string
  name: string
}
```

Dalam komponen:

```tsx
function UserPage() {
  const user = Route.useLoaderData()

  return <div>{user.name}</div>
}
```

Sekali lagi, return type loader bergerak terus ke hook.

---

# 31. Loader boleh memanggil function lain

```ts
async function getUser() {
  return {
    id: 1,
    name: 'Ali',
  }
}
```

Kemudian:

```tsx
export const Route = createFileRoute('/')({
  loader: async () => {
    return getUser()
  },
  component: Home,
})
```

`getUser()` memulangkan:

```ts
Promise<{
  id: number
  name: string
}>
```

Loader juga menjadi async dan memulangkan Promise.

TanStack Router membuka Promise itu.

Jadi:

```ts
const user = Route.useLoaderData()
```

mempunyai type:

```ts
{
  id: number
  name: string
}
```

Bukan Promise.

---

# 32. Promise berlapis tidak menjadi masalah

Dalam JavaScript, `async` akan meratakan Promise.

Contohnya:

```ts
async function loader() {
  return Promise.resolve({
    name: 'Ali',
  })
}
```

Return type praktikalnya tetap:

```ts
Promise<{
  name: string
}>
```

TanStack Router mendapatkan resolved data type:

```ts
{
  name: string
}
```

Dalam TypeScript moden, utility type seperti `Awaited<T>` sering digunakan untuk membuka Promise dengan lebih lengkap.

Secara konsep:

```ts
type Data = Awaited<ReturnType<typeof loader>>
```

Hasilnya ialah data sebenar selepas Promise selesai.

---

# 33. Bezakan return type loader dan nilai hook

Untuk loader async:

```ts
const loader = async () => ({
  name: 'Ali',
})
```

Return type function:

```ts
Promise<{
  name: string
}>
```

Tetapi resolved loader data type:

```ts
{
  name: string
}
```

Dan return type hook:

```ts
{
  name: string
}
```

Jadi:

```text
ReturnType<typeof loader>
=
Promise<Data>
```

Tetapi:

```text
Route.useLoaderData()
=
Data
```

---

# 34. Latihan dalam editor

Buka:

```text
src/routes/index.tsx
```

Tambahkan loader:

```tsx
export const Route = createFileRoute('/')({
  loader: () => ({
    name: 'Ali',
    age: 30,
  }),
  component: Home,
})
```

Dalam `Home`:

```tsx
function Home() {
  const data = Route.useLoaderData()

  return <div>{data.name}</div>
}
```

Hover pada:

```ts
data
```

TypeScript sepatutnya menunjukkan type seperti:

```ts
{
  name: string
  age: number
}
```

---

# 35. Tukar loader kepada async

Ubah:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

kepada:

```ts
loader: async () => ({
  name: 'Ali',
  age: 30,
})
```

Hover pada `data` sekali lagi.

Type tetap:

```ts
{
  name: string
  age: number
}
```

Ia tidak berubah menjadi Promise.

Ini membuktikan bahawa Promise telah dibuka pada tahap type.

---

# 36. Cuba property yang tidak wujud

Tambahkan:

```ts
data.bogus
```

TypeScript akan menghasilkan ralat seperti:

```text
Property 'bogus' does not exist on type
'{ name: string; age: number; }'.
```

Mesej ini memberitahu anda bahawa TypeScript mengetahui property yang sah hanyalah:

```ts
name
age
```

Ini ialah bukti type flow sedang berfungsi.

---

# 37. Cuba ubah bentuk loader

Ubah loader kepada:

```ts
loader: () => ({
  title: 'Halaman utama',
  visits: 100,
})
```

Sekarang `data` menjadi:

```ts
{
  title: string
  visits: number
}
```

Kod ini tidak lagi sah:

```ts
data.name
```

Kod baru yang sah:

```ts
data.title
data.visits
```

Komponen berubah secara type-safe bersama loader.

---

# 38. Ralat loader membantu mengesan perubahan API

Bayangkan loader asal memulangkan:

```ts
{
  user: {
    name: string
  }
}
```

Komponen menggunakan:

```ts
data.user.name
```

Kemudian loader diubah menjadi:

```ts
{
  profile: {
    displayName: string
  }
}
```

TypeScript akan menandakan semua tempat yang masih menggunakan:

```ts
data.user.name
```

Ini sangat berguna semasa refactoring.

Anda tidak perlu mencari penggunaan lama secara manual sahaja.

Compiler membantu menunjukkan setiap tempat yang tidak lagi sepadan.

---

# 39. Kenapa anotasi manual biasanya tidak diperlukan

Anda boleh menulis:

```ts
type HomeLoaderData = {
  name: string
  age: number
}
```

Kemudian:

```ts
loader: (): HomeLoaderData => ({
  name: 'Ali',
  age: 30,
})
```

Ini sah, tetapi untuk data mudah ia mungkin berlebihan.

TanStack Router direka supaya inferens dapat membawa type tersebut secara automatik.

Versi lebih ringkas:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

sudah cukup.

Gunakan anotasi manual apabila ia menambah nilai, contohnya:

* type dikongsi oleh beberapa function,
* data datang daripada API,
* anda mahu menetapkan kontrak yang jelas,
* return type sangat kompleks,
* atau anda mahu memastikan function mematuhi interface tertentu.

---

# 40. Jangan anotasi `useLoaderData()` secara paksa

Elakkan:

```ts
const data = Route.useLoaderData() as {
  name: string
  age: number
}
```

Ini tidak diperlukan jika loader sudah bertype dengan betul.

Lebih buruk, assertion itu boleh menjadi salah apabila loader berubah.

Contohnya loader berubah kepada:

```ts
loader: () => ({
  username: 'ali',
})
```

Tetapi assertion lama masih berkata:

```ts
{
  name: string
  age: number
}
```

Anda telah memutuskan type flow dan menggantikannya dengan janji manual yang mungkin palsu.

Lebih baik biarkan TypeScript mendapat type daripada loader sebenar.

---

# 41. Rantaian type yang perlu diingati

```text
Nilai yang dipulangkan loader
        ↓
TypeScript mengesan return type
        ↓
Jika Promise, resolved type diambil
        ↓
Type itu disimpan sebagai loaderData route
        ↓
Route.useLoaderData() membaca loaderData
        ↓
Komponen mendapat data bertype
```

Versi kod:

```ts
loader: async () => ({
  name: 'Ali',
  age: 30,
})
```

menghasilkan:

```ts
Route.useLoaderData()
```

dengan type:

```ts
{
  name: string
  age: number
}
```

---

# 42. Kuiz ringkas

## Soalan 1

Jika loader ialah async:

```ts
loader: async () => ({
  name: 'Ali',
})
```

apakah type yang dipulangkan oleh:

```ts
Route.useLoaderData()
```

Pilihan:

* `Promise` kepada data
* Data itu sendiri
* `any`
* `unknown`

Jawapan:

```text
Data itu sendiri
```

TypeScript membuka Promise dan mengambil resolved value type.

---

## Soalan 2

Kenapa `Route.useLoaderData()` mengetahui type data?

Pilihan:

* Runtime magic
* Global registry rawak
* Type `any`
* Type milik object Route

Jawapan:

```text
Type milik object Route
```

Object `Route` membawa type loaderData daripada loader route tersebut.

---

## Soalan 3

Dari mana datangnya type `loaderData`?

Pilihan:

* Return type loader
* Props komponen
* Global state
* URL string

Jawapan:

```text
Return type loader
```

TypeScript membuat inferens daripada nilai yang dipulangkan loader.

---

## Soalan 4

Apakah fungsi `infer` dalam type ini?

```ts
T extends (...args: any[]) => infer TReturn
  ? TReturn
  : never
```

Jawapan:

```text
Mengambil dan menamakan return type function sebagai TReturn.
```

---

## Soalan 5

Apakah hasil type berikut?

```ts
type Result =
  Promise<string> extends Promise<infer Value>
    ? Value
    : never
```

Jawapan:

```ts
string
```

`Value` diinfer sebagai `string`.

---

# 43. Glosari ringkas

## Loader

Function route yang mendapatkan atau menyediakan data sebelum komponen route digunakan.

```ts
loader: () => data
```

---

## Return type

Type nilai yang dipulangkan oleh function.

```ts
() => number
```

Return typenya ialah `number`.

---

## Loader data

Data yang telah dihasilkan oleh loader dan tersedia kepada route.

---

## Type inference

Proses TypeScript menyimpulkan type daripada kod yang ditulis.

---

## Conditional type

Type yang memilih hasil berdasarkan syarat type.

```ts
T extends U ? X : Y
```

---

## `infer`

Kata kunci yang mengambil sebahagian type dan memberinya nama sementara dalam conditional type.

---

## Promise unwrapping

Proses mendapatkan resolved value type daripada:

```ts
Promise<T>
```

Hasilnya ialah:

```ts
T
```

---

## `never`

Type yang mewakili sesuatu yang tidak pernah berlaku atau tiada hasil type yang sah.

Dalam conditional type, ia sering digunakan sebagai hasil apabila syarat tidak dipenuhi.

---

# 44. Ringkasan utama

Apabila anda menulis:

```tsx
export const Route = createFileRoute('/')({
  loader: async () => ({
    name: 'Ali',
    age: 30,
  }),
  component: Home,
})
```

TypeScript melakukan perkara berikut:

1. Mengesan return type loader.
2. Mengetahui bahawa loader async memulangkan Promise.
3. Membuka Promise kepada resolved data type.
4. Menyimpan type itu dalam type object `Route`.
5. Memberikan type yang sama kepada `Route.useLoaderData()`.
6. Menyediakan autocomplete dan pemeriksaan property dalam komponen.

Formula utamanya:

```text
Loader return type
+ Promise unwrapping
+ Route type
= Typed useLoaderData()
```

Versi paling ringkas:

```text
Anda tulis data sekali
→ TypeScript membawa typenya ke hadapan
```

Contohnya:

```ts
loader: () => ({
  name: 'Ali',
  age: 30,
})
```

menghasilkan:

```ts
const data = Route.useLoaderData()
```

dengan type:

```ts
{
  name: string
  age: number
}
```

Tiada `any`, tiada casting, dan tiada anotasi berulang.

Pelajaran seterusnya akan menerangkan bagaimana route params dan search params dibaca dengan type safety penuh.
