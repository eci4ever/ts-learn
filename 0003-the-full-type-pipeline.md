# Pelajaran 3 · TypeScript untuk TanStack

## Keseluruhan Rantaian Type: Dari `'/'` Hingga ke `<Link>`

**Misi:** Membaca semula kod TanStack Router yang sebelum ini kelihatan rumit, tetapi kali ini dengan semua istilah asas yang diperlukan.

Dalam pelajaran sebelumnya, kita telah belajar beberapa konsep secara berasingan. Sekarang kita akan menyambungkan semuanya menjadi satu aliran lengkap.

Kita mahu memahami bagaimana nilai kecil ini:

```ts
'/'
```

akhirnya membolehkan TypeScript memeriksa kod seperti:

```tsx
<Link to="/" />
```

dan menolak:

```tsx
<Link to="/benda-lain" />
```

Keseluruhan rantaian itu bermula daripada route file, bergerak ke generated route tree, didaftarkan kepada router, kemudian digunakan oleh semua komponen dan hook TanStack Router.

---

# 1. Lima asas dalam satu gambaran

Lima konsep berikut ialah asas kepada keseluruhan type safety TanStack Router.

## Literal type

Nilai:

```ts
'/'
```

boleh menjadi type yang sangat tepat:

```ts
'/'
```

Ia bukan semestinya type umum:

```ts
string
```

Literal type `'/'` hanya mempunyai satu nilai yang sah, iaitu `'/'`.

---

## Generic

Generic ialah function, class atau type yang mempunyai type parameter.

Contoh:

```ts
function identity<T>(value: T): T {
  return value
}
```

`T` ialah type parameter yang boleh berubah pada setiap panggilan.

```ts
identity('hello')
// T = string

identity(123)
// T = number
```

Dalam TanStack Router, `createFileRoute` juga ialah generic function.

---

## Generic constraint

Generic constraint mengehadkan type yang boleh digunakan oleh generic.

Contohnya:

```ts
function printText<T extends string>(value: T) {
  console.log(value)
}
```

`T extends string` bermaksud `T` mestilah sejenis string.

Kod ini sah:

```ts
printText('hello')
```

Kod ini tidak sah:

```ts
printText(123)
```

Dalam TanStack Router, constraint memastikan path mestilah salah satu route yang benar-benar wujud.

---

## Type argument inference

TypeScript boleh menyimpulkan generic type daripada nilai yang diberikan.

Contohnya:

```ts
identity('hello')
```

Kita tidak perlu menulis:

```ts
identity<string>('hello')
```

TypeScript melihat nilai `'hello'` lalu menyimpulkan type `T`.

Dalam TanStack Router:

```ts
createFileRoute('/')
```

TypeScript menyimpulkan:

```ts
TFilePath = '/'
```

---

## Declaration merging

Declaration merging berlaku apabila TypeScript menggabungkan beberapa declaration bagi interface yang sama.

Contohnya:

```ts
interface User {
  name: string
}
```

Kemudian:

```ts
interface User {
  email: string
}
```

TypeScript menganggap hasil akhirnya seperti:

```ts
interface User {
  name: string
  email: string
}
```

TanStack Router menggunakan mekanisme ini untuk memasukkan route app anda ke dalam type library.

---

# 2. Bermula di `src/routes/index.tsx`

Buka fail:

```text
src/routes/index.tsx
```

Anda akan melihat kod seperti ini:

```ts
export const Route = createFileRoute('/')({
  component: Home,
})
```

Kod ini pendek, tetapi banyak perkara berlaku pada tahap type.

Mari ikut alirannya satu demi satu.

---

# 3. `createFileRoute` ialah generic function

Signature ringkasnya kelihatan seperti ini:

```ts
createFileRoute<
  TFilePath extends keyof FileRoutesByPath
>(
  path?: TFilePath
)
```

Kita pecahkan kepada beberapa bahagian.

---

## Type parameter

```ts
TFilePath
```

ialah type parameter.

Ia bertindak seperti pemboleh ubah untuk type path.

Pada panggilan berbeza, nilainya boleh berbeza.

Contohnya secara konsep:

```ts
createFileRoute('/')
// TFilePath = '/'

createFileRoute('/users')
// TFilePath = '/users'

createFileRoute('/posts/$postId')
// TFilePath = '/posts/$postId'
```

---

## Constraint

Bahagian ini:

```ts
TFilePath extends keyof FileRoutesByPath
```

bermaksud:

> `TFilePath` mestilah salah satu key yang terdapat dalam `FileRoutesByPath`.

Jika `FileRoutesByPath` mengandungi:

```ts
interface FileRoutesByPath {
  '/': {}
  '/users': {}
  '/settings': {}
}
```

maka:

```ts
keyof FileRoutesByPath
```

menghasilkan:

```ts
'/' | '/users' | '/settings'
```

Jadi `TFilePath` hanya boleh menjadi salah satu daripada tiga nilai itu.

---

# 4. Nilai `'/'` ialah literal type

Dalam panggilan:

```ts
createFileRoute('/')
```

argument yang dihantar ialah:

```ts
'/'
```

TypeScript tidak semestinya melihatnya sebagai `string`.

Dalam konteks generic inference, TypeScript boleh mengekalkan nilai tersebut sebagai literal type:

```ts
'/'
```

Jadi TypeScript mempunyai maklumat yang sangat tepat.

Ia tahu bahawa function ini bukan menerima sebarang string.

Ia menerima path tertentu:

```ts
'/'
```

---

# 5. TypeScript melakukan inference

Kita tidak menulis:

```ts
createFileRoute<'/'>('/')
```

Namun secara konsep, itulah yang disimpulkan oleh TypeScript.

Daripada argument:

```ts
'/'
```

TypeScript menetapkan:

```ts
TFilePath = '/'
```

Proses ini dipanggil **type argument inference**.

Alirannya:

```text
Argument yang diterima: '/'
        ↓
Type literal argument: '/'
        ↓
Type parameter disimpulkan
        ↓
TFilePath = '/'
```

---

# 6. Constraint diperiksa

Selepas menyimpulkan:

```ts
TFilePath = '/'
```

TypeScript perlu memastikan bahawa type itu memenuhi constraint:

```ts
TFilePath extends keyof FileRoutesByPath
```

Secara konsep, ia memeriksa:

```ts
'/' extends keyof FileRoutesByPath
```

Jika `'/'` memang salah satu key dalam `FileRoutesByPath`, panggilan itu sah.

Jika tidak, TypeScript menghasilkan ralat.

Contohnya:

```ts
createFileRoute('/benda-lain')
```

akan menjadi masalah jika `'/benda-lain'` tidak terdapat dalam:

```ts
keyof FileRoutesByPath
```

---

# 7. Return type menyimpan path tersebut

Return type `createFileRoute` membawa `TFilePath` ke dalam object route.

Secara ringkas, bayangkan return type seperti ini:

```ts
FileRoute<TFilePath>
```

Selepas inference:

```ts
FileRoute<'/'>
```

Jadi object `Route` menyimpan maklumat bahawa ia berkaitan dengan route `'/'`.

Contohnya:

```ts
Route.fullPath
```

mempunyai type:

```ts
'/'
```

Bukan:

```ts
string
```

Ini penting kerana object `Route` sekarang mempunyai identiti type yang tepat.

Ia bukan sekadar “satu route”.

Ia ialah:

> Route untuk path `'/'`.

---

# 8. Dari mana datangnya `FileRoutesByPath`?

Sekarang timbul soalan penting:

> Bagaimana TypeScript tahu bahawa `FileRoutesByPath` mempunyai key `'/'`?

Jawapannya ada dalam:

```text
src/routeTree.gen.ts
```

Fail ini dijana secara automatik oleh TanStack Router.

Di dalamnya, terdapat module augmentation yang lebih kurang seperti ini:

```ts
declare module '@tanstack/react-router' {
  interface FileRoutesByPath {
    '/': {
      id: '/'
      path: '/'
      fullPath: '/'
      parentRoute: typeof rootRouteImport
    }
  }
}
```

---

# 9. Memahami `declare module`

Bahagian:

```ts
declare module '@tanstack/react-router'
```

bermaksud:

> Buka semula declaration type bagi module `@tanstack/react-router`.

Ia tidak mencipta module baru pada runtime.

Ia hanya menambah maklumat pada sistem type TypeScript.

Ini dipanggil **module augmentation**.

---

# 10. Memahami declaration merging

Library mungkin mempunyai interface asas seperti:

```ts
interface FileRoutesByPath {}
```

Kemudian generated file menambah:

```ts
interface FileRoutesByPath {
  '/': {
    id: '/'
    path: '/'
    fullPath: '/'
  }
}
```

TypeScript menggabungkan kedua-duanya.

Hasil akhirnya, dari sudut pandangan TypeScript:

```ts
interface FileRoutesByPath {
  '/': {
    id: '/'
    path: '/'
    fullPath: '/'
  }
}
```

Proses ini dipanggil **declaration merging**.

---

# 11. Hasil daripada `keyof FileRoutesByPath`

Jika hanya ada satu route:

```ts
interface FileRoutesByPath {
  '/': {}
}
```

maka:

```ts
keyof FileRoutesByPath
```

ialah:

```ts
'/'
```

Jika terdapat lebih banyak route:

```ts
interface FileRoutesByPath {
  '/': {}
  '/users': {}
  '/settings': {}
}
```

maka:

```ts
keyof FileRoutesByPath
```

ialah:

```ts
'/' | '/users' | '/settings'
```

Ini dipanggil union of literal types.

Maksudnya path yang sah hanyalah salah satu daripada nilai tersebut.

---

# 12. `routeTree.gen.ts` ialah peta route bertype

Anggap `routeTree.gen.ts` sebagai jambatan antara:

```text
Struktur fail route dalam projek
```

dan:

```text
Sistem type TypeScript
```

Contohnya, anda mempunyai fail:

```text
src/routes/index.tsx
src/routes/users.tsx
src/routes/settings.tsx
```

TanStack Router menjana type map seperti:

```ts
interface FileRoutesByPath {
  '/': {}
  '/users': {}
  '/settings': {}
}
```

Disebabkan itu, TypeScript mengetahui route yang benar-benar wujud.

---

# 13. Kenapa `routeTree.gen.ts` tidak patut diedit manual?

Nama fail tersebut mempunyai `.gen`, yang bermaksud generated.

Ia dibina semula oleh plugin atau CLI TanStack Router berdasarkan fail dalam folder routes.

Jika anda mengedit secara manual:

```ts
interface FileRoutesByPath {
  '/benda-palsu': {}
}
```

perubahan tersebut boleh hilang apabila route tree dijana semula.

Sebab itu jawapan kepada soalan:

> Kenapa `routeTree.gen.ts` tidak boleh diedit secara manual?

ialah:

```text
Kerana ia akan ditimpa semula.
```

Route sebenar sepatutnya ditambah melalui fail dalam:

```text
src/routes
```

bukan dengan mengubah generated file.

---

# 14. Peranan `src/router.tsx`

Seterusnya buka:

```text
src/router.tsx
```

Anda mungkin melihat declaration seperti ini:

```ts
declare module '@tanstack/react-router' {
  interface Register {
    router: ReturnType<typeof getRouter>
  }
}
```

Mekanismenya sama seperti `FileRoutesByPath`.

Module `@tanstack/react-router` dibuka semula, kemudian interface `Register` ditambah maklumat.

---

# 15. Apakah fungsi interface `Register`?

Interface `Register` memberitahu TanStack Router:

> Inilah type router sebenar yang digunakan oleh aplikasi ini.

Kod:

```ts
router: ReturnType<typeof getRouter>
```

bermaksud type `router` diambil daripada return type function:

```ts
getRouter
```

Jika `getRouter()` mengembalikan router yang mempunyai seluruh route tree aplikasi, TypeScript boleh mengesan semua maklumat itu.

Contohnya:

* route yang sah,
* path params,
* search params,
* loader data,
* context,
* navigation options.

---

# 16. Memahami `ReturnType`

`ReturnType` ialah utility type TypeScript.

Contohnya:

```ts
function getUser() {
  return {
    id: 1,
    name: 'Nik',
  }
}
```

Kita boleh mendapatkan type hasil function itu:

```ts
type User = ReturnType<typeof getUser>
```

Hasilnya lebih kurang:

```ts
type User = {
  id: number
  name: string
}
```

Dalam router:

```ts
ReturnType<typeof getRouter>
```

bermaksud:

> Gunakan type router sebenar yang dipulangkan oleh `getRouter`.

---

# 17. Bagaimana `Register` membantu `<Link>`?

Komponen `Link` datang daripada library:

```ts
import { Link } from '@tanstack/react-router'
```

Library itu tidak boleh mengetahui route aplikasi anda hanya daripada package asal.

Ia tidak tahu sama ada app anda mempunyai:

```text
/users
/settings
/posts
```

Tetapi selepas anda menambah:

```ts
interface Register {
  router: ReturnType<typeof getRouter>
}
```

library boleh membaca type router aplikasi anda.

Daripada type router tersebut, `Link` boleh mendapatkan senarai path yang sah.

Secara konsep:

```ts
type ValidPaths =
  '/' |
  '/users' |
  '/settings'
```

Kemudian prop `to` boleh diperiksa.

---

# 18. Kenapa `<Link to="/">` sah?

Apabila anda menulis:

```tsx
<Link to="/">Home</Link>
```

TypeScript memeriksa nilai:

```ts
'/'
```

terhadap senarai route yang sah.

Jika type route ialah:

```ts
'/' | '/users' | '/settings'
```

maka `'/'` diterima.

Secara konsep:

```ts
'/' extends ValidPaths
```

Jawapannya ialah ya.

Jadi tiada ralat.

---

# 19. Kenapa `<Link to="/benda-lain">` menjadi ralat?

Apabila anda menulis:

```tsx
<Link to="/benda-lain">Benda lain</Link>
```

TypeScript memeriksa:

```ts
'/benda-lain'
```

terhadap:

```ts
'/' | '/users' | '/settings'
```

Nilai itu tidak sepadan dengan mana-mana route.

Jadi TypeScript menghasilkan ralat sebelum aplikasi dijalankan.

Ini ialah compile-time checking atau pemeriksaan semasa pembangunan.

Ia bukan semata-mata pemeriksaan runtime.

---

# 20. Keseluruhan aliran dari route file ke `<Link>`

Sekarang kita boleh melihat seluruh rantaian.

## Langkah 1: Route file ditulis

```ts
createFileRoute('/')
```

Nilai `'/'` ialah literal type.

---

## Langkah 2: Generic menangkap path

```ts
TFilePath = '/'
```

Ini berlaku melalui type argument inference.

---

## Langkah 3: Constraint memastikan path sah

```ts
TFilePath extends keyof FileRoutesByPath
```

TypeScript memastikan `'/'` berada dalam route map.

---

## Langkah 4: Generated file menyediakan route map

```ts
interface FileRoutesByPath {
  '/': {}
}
```

Ini berlaku melalui module augmentation dan declaration merging.

---

## Langkah 5: Object route menyimpan path

```ts
Route.fullPath
```

mempunyai type:

```ts
'/'
```

---

## Langkah 6: Router didaftarkan

```ts
interface Register {
  router: ReturnType<typeof getRouter>
}
```

TanStack Router kini mengetahui type router app anda.

---

## Langkah 7: `Link` membaca type router

```tsx
<Link to="/" />
```

`to` diperiksa terhadap semua route yang sah.

---

## Langkah 8: Path tidak sah ditolak

```tsx
<Link to="/benda-lain" />
```

menghasilkan ralat TypeScript jika route tersebut tidak wujud.

---

# 21. Gambaran rantaian dalam bentuk diagram

```text
src/routes/index.tsx
        │
        │ createFileRoute('/')
        ▼
Literal type '/'
        │
        │ Type argument inference
        ▼
TFilePath = '/'
        │
        │ Generic constraint
        ▼
keyof FileRoutesByPath
        │
        │ Declaration merging
        ▼
src/routeTree.gen.ts
        │
        │ Typed route tree
        ▼
getRouter()
        │
        │ ReturnType<typeof getRouter>
        ▼
interface Register
        │
        │ Library membaca router app
        ▼
<Link to="...">
        │
        ▼
Path diperiksa oleh TypeScript
```

---

# 22. Bagaimana hook turut mendapat type?

Mekanisme ini bukan hanya untuk `Link`.

Maklumat route yang sama digunakan oleh hook seperti:

```ts
Route.useParams()
Route.useSearch()
Route.useLoaderData()
```

Contohnya, jika route anda ialah:

```text
/posts/$postId
```

TanStack Router boleh mengetahui bahawa route tersebut mempunyai param:

```ts
postId
```

Jadi:

```ts
const params = Route.useParams()
```

boleh menghasilkan type seperti:

```ts
{
  postId: string
}
```

Begitu juga jika route mempunyai loader:

```ts
loader: async () => {
  return {
    message: 'Hello',
  }
}
```

maka:

```ts
const data = Route.useLoaderData()
```

boleh mengetahui bahawa:

```ts
data.message
```

ialah `string`.

Semua ini masih berpunca daripada route identity yang bermula dengan literal path.

---

# 23. Kuiz pengukuhan

## Soalan 1

Dalam:

```ts
createFileRoute('/')
```

apakah type bagi `TFilePath`?

Pilihan:

* `string`
* `'/'`
* `any`
* `unknown`

Jawapan:

```ts
'/'
```

TypeScript menyimpulkannya daripada literal argument `'/'`.

---

## Soalan 2

Bagaimana:

```ts
keyof FileRoutesByPath
```

mendapat nilai `'/'`?

Pilihan:

* Ditulis manual dalam setiap component
* Dihasilkan semasa runtime
* Daripada `any`
* Declaration merging

Jawapan:

```text
Declaration merging
```

Generated file menambah key `'/'` kepada interface `FileRoutesByPath`.

---

## Soalan 3

Kenapa kod berikut menjadi ralat?

```tsx
<Link to="/benda-lain" />
```

Pilihan:

* Disebabkan type router
* Sebab ia string
* Disebabkan pemeriksaan runtime
* Disebabkan CSS

Jawapan:

```text
Disebabkan type router.
```

Lebih tepat, `'/benda-lain'` tidak terdapat dalam senarai path yang diketahui oleh typed router.

---

## Soalan 4

Kenapa `routeTree.gen.ts` tidak patut diedit secara manual?

Pilihan:

* Kerana ia rahsia
* Kerana fail terlalu besar
* Kerana ia akan ditimpa
* Kerana menggunakan `any`

Jawapan:

```text
Kerana ia akan ditimpa.
```

Fail itu dijana semula daripada route files dalam projek.

---

# 24. Cuba dalam editor

Buka:

```text
src/routes/index.tsx
```

Hover pada:

```ts
Route
```

Perhatikan type yang dipaparkan.

Anda sepatutnya dapat melihat path literal `'/'` terkandung dalam type `FileRoute`.

Contohnya lebih kurang:

```ts
FileRoute<'/', ...>
```

Kemudian dalam component, cuba:

```tsx
<Link to="/">Home</Link>
```

Tiada ralat sepatutnya muncul.

Sekarang tukar kepada:

```tsx
<Link to="/benda-lain">Home</Link>
```

Jika route tersebut tidak wujud, TypeScript akan menunjukkan ralat dengan segera.

Kemudian tukar semula kepada:

```tsx
<Link to="/">Home</Link>
```

Eksperimen kecil ini membuktikan bahawa `Link` membaca senarai route daripada type router aplikasi.

---

# 25. Perkara penting yang perlu dibezakan

## Route tidak dikesan melalui runtime sahaja

TanStack Router memang menggunakan route pada runtime untuk memaparkan halaman.

Tetapi ralat pada:

```tsx
<Link to="/benda-lain" />
```

boleh muncul sebelum aplikasi dijalankan.

Itu bermaksud pemeriksaan tersebut datang daripada sistem type.

---

## `Link` tidak membaca folder secara langsung

Komponen `Link` tidak membuka folder:

```text
src/routes
```

Ia membaca type router yang telah dibina daripada generated declarations.

Alirannya ialah:

```text
Route files
→ generated route tree
→ router type
→ Register
→ Link
```

---

## `Register` dan `FileRoutesByPath` mempunyai tugas berbeza

`FileRoutesByPath` menyimpan peta typed untuk file routes.

```ts
interface FileRoutesByPath {
  '/': {}
}
```

`Register` pula memberitahu library type router sebenar untuk aplikasi.

```ts
interface Register {
  router: ReturnType<typeof getRouter>
}
```

Kedua-duanya menggunakan module augmentation, tetapi tujuan mereka berbeza.

---

# 26. Ringkasan keseluruhan

Apabila anda menulis:

```ts
export const Route = createFileRoute('/')({
  component: Home,
})
```

perkara berikut berlaku:

1. `'/'` dibaca sebagai literal type.
2. `createFileRoute` menangkapnya melalui generic.
3. TypeScript menyimpulkan `TFilePath = '/'`.
4. Generic constraint memastikan `'/'` ialah key yang sah.
5. `routeTree.gen.ts` menyediakan key tersebut melalui declaration merging.
6. Object `Route` menyimpan path tepat `'/'`.
7. `getRouter()` menggabungkan keseluruhan route tree.
8. Interface `Register` mendaftarkan type router kepada library.
9. `Link`, navigation dan hooks membaca type router itu.
10. Path yang tidak wujud menghasilkan ralat TypeScript.

Formula lengkapnya:

```text
Literal type
+ Generic
+ Generic constraint
+ Type inference
+ Declaration merging
+ Router registration
= Type-safe navigation
```

Versi yang lebih ringkas:

```text
'/'
→ TFilePath
→ FileRoutesByPath
→ Route
→ Router
→ Register
→ Link
```

Itulah keseluruhan rantaian type daripada nilai `'/'` hingga ke:

```tsx
<Link to="/" />
```

Pelajaran seterusnya akan memberi fokus kepada cara membaca ralat TypeScript sebenar dalam repository, termasuk mengenal pasti bahagian ralat yang penting dan bahagian yang boleh diketepikan buat sementara waktu.
