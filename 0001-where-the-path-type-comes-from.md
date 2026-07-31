# Pelajaran 1 · TypeScript untuk TanStack

## Dari mana datangnya jenis `path`?

**Misi:** Belajar membaca codebase ini dengan lebih yakin.

Pelajaran pertama ini menerangkan asas utama yang menjadikan sistem type safety TanStack Router berfungsi — iaitu mekanisme yang membolehkan:

```ts
createFileRoute('/')
```

“mengetahui” bahawa route tersebut ialah halaman utama.

Buka fail:

```text
src/routes/index.tsx
```

Pada baris 3, keseluruhan route ditulis seperti ini:

```ts
export const Route = createFileRoute('/')({
  component: Home,
})
```

Perhatikan nilai `'/'`.

Bagaimana TypeScript tahu bahawa:

```ts
Route.fullPath
```

mempunyai type tepat:

```ts
'/'
```

dan bukannya:

```ts
string
```

atau:

```ts
any
```

Sekiranya TypeScript hanya menganggap path tersebut sebagai `string`, maka perkara seperti ini tidak lagi selamat:

```tsx
<Link to="/" />
```

TypeScript juga mungkin menerima path yang tidak wujud:

```tsx
<Link to="/nonsense" />
```

Begitu juga dengan hook seperti:

```ts
Route.useLoaderData()
```

TypeScript tidak akan tahu loader untuk route mana yang sedang digunakan.

Jawapannya boleh diringkaskan kepada satu ayat:

> Literal type + generic function + type inference.

---

## 1. Literal type: `'/'` bukan sekadar `string`

Dalam TypeScript, nilai `'/'` boleh mempunyai type yang lebih khusus daripada `string`.

Terdapat satu type yang hanya membenarkan satu nilai sahaja:

```ts
'/'
```

Type ini dipanggil **literal type**.

Literal type `'/'` ialah subtype kepada `string`.

Maksudnya:

* Setiap `'/'` ialah string.
* Tetapi bukan setiap string ialah `'/'`.

Contohnya:

```ts
const home = '/'
```

TypeScript akan mengesan type `home` sebagai:

```ts
'/'
```

Bukan `string`.

Sebaliknya:

```ts
let anything = '/'
```

TypeScript biasanya akan mengesan type tersebut sebagai:

```ts
string
```

Perbezaannya ialah `const` tidak boleh ditukar selepas diisytiharkan.

```ts
const home = '/'
```

Nilainya akan kekal `'/'`, jadi TypeScript boleh menyimpan type yang sangat tepat.

Tetapi `let` boleh ditukar:

```ts
let anything = '/'

anything = '/dashboard'
anything = '/users'
```

Disebabkan nilainya boleh berubah, TypeScript akan meluaskan type tersebut daripada `'/'` kepada `string`.

Proses ini dipanggil **type widening**.

---

## 2. Generic function: menangkap type path

Generic function ialah function yang mempunyai type parameter.

Type parameter biasanya ditulis menggunakan tanda kurungan sudut:

```ts
<T>
```

Type parameter ini bertindak seperti pemboleh ubah untuk type.

Contoh generic function mudah:

```ts
function identity<T>(value: T): T {
  return value
}
```

Apabila dipanggil seperti ini:

```ts
identity('hello')
```

TypeScript mengesan bahawa `T` ialah `string`.

Dalam TanStack Router, signature sebenar `createFileRoute` lebih kurang seperti ini:

```ts
declare function createFileRoute<
  TFilePath extends keyof FileRoutesByPath
>(
  path?: TFilePath
): FileRoute<TFilePath, ...>['createRoute']
```

Mari baca bahagian demi bahagian.

### Bahagian pertama

```ts
<TFilePath extends keyof FileRoutesByPath>
```

Maksudnya:

> Function ini mempunyai satu type parameter bernama `TFilePath`.

Kemudian:

```ts
extends keyof FileRoutesByPath
```

bermaksud:

> `TFilePath` mestilah salah satu key yang terdapat dalam `FileRoutesByPath`.

Contohnya, sekiranya `FileRoutesByPath` mengandungi:

```ts
interface FileRoutesByPath {
  '/': {}
  '/users': {}
  '/settings': {}
}
```

Maka:

```ts
keyof FileRoutesByPath
```

akan menjadi union type:

```ts
'/' | '/users' | '/settings'
```

Jadi `TFilePath` hanya boleh menjadi salah satu daripada path tersebut.

---

### Bahagian kedua

```ts
(path?: TFilePath)
```

Maksudnya argument `path` menggunakan type yang sama, iaitu `TFilePath`.

Apabila anda menulis:

```ts
createFileRoute('/')
```

TypeScript melihat nilai literal `'/'` dan menetapkan:

```ts
TFilePath = '/'
```

Kemudian return type function tersebut menggunakan kembali `TFilePath`:

```ts
FileRoute<TFilePath, ...>
```

Selepas type parameter digantikan, ia secara konsep menjadi:

```ts
FileRoute<'/', ...>
```

Jadi literal type `'/'` disimpan dan dibawa ke seluruh object `Route`.

Sebab itulah TypeScript boleh mengetahui bahawa:

```ts
Route.fullPath
```

mempunyai type:

```ts
'/'
```

dan bukannya hanya:

```ts
string
```

---

## 3. Type inference: anda tidak perlu menulis generic sendiri

Secara manual, anda sebenarnya boleh membayangkan pemanggilan function tersebut seperti ini:

```ts
createFileRoute<'/'>('/')
```

Tetapi dalam kod sebenar, anda hanya menulis:

```ts
createFileRoute('/')
```

TypeScript mengisi type argument tersebut secara automatik.

Proses ini dipanggil:

```text
Type argument inference
```

atau **inferens argument type**.

TypeScript melihat nilai argument:

```ts
'/'
```

Kemudian ia membuat kesimpulan bahawa:

```ts
TFilePath
```

ialah:

```ts
'/'
```

Inferens ini boleh berlaku kerana type parameter `TFilePath` digunakan pada kedudukan argument:

```ts
path?: TFilePath
```

Secara ringkas:

```ts
createFileRoute('/')
```

menyebabkan TypeScript berfikir seperti ini:

```ts
Nilai yang diterima ialah '/'
Jadi TFilePath ialah '/'
Return type mesti menggunakan '/'
```

---

## 4. Dari mana datangnya `FileRoutesByPath`?

Dalam signature tadi terdapat constraint:

```ts
TFilePath extends keyof FileRoutesByPath
```

Persoalannya, dari mana datangnya interface `FileRoutesByPath`?

Buka fail:

```text
src/routeTree.gen.ts
```

Di dalam fail tersebut terdapat kod yang lebih kurang seperti ini:

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

Bahagian ini dipanggil **module augmentation**.

Ia membuka semula module:

```ts
'@tanstack/react-router'
```

Kemudian ia menambah maklumat baru kepada interface:

```ts
FileRoutesByPath
```

Teknik menggabungkan declaration interface ini dipanggil:

```text
Declaration merging
```

Library TanStack Router mungkin menyediakan interface asas:

```ts
interface FileRoutesByPath {}
```

Kemudian fail generated anda menambah route sebenar:

```ts
interface FileRoutesByPath {
  '/': {
    id: '/'
    path: '/'
    fullPath: '/'
  }
}
```

TypeScript akan menggabungkan kedua-dua declaration tersebut.

Hasilnya, TypeScript tahu bahawa route `'/'` benar-benar wujud.

---

## Kenapa fail `routeTree.gen.ts` penting?

Fail ini biasanya dijana secara automatik oleh TanStack Router.

Ia membaca struktur fail route seperti:

```text
src/routes/index.tsx
src/routes/users.tsx
src/routes/settings.tsx
```

Kemudian ia membina typed route map.

Contohnya:

```ts
interface FileRoutesByPath {
  '/': {}
  '/users': {}
  '/settings': {}
}
```

Disebabkan itu, kod ini sah:

```tsx
<Link to="/" />
```

Kod ini juga sah sekiranya route tersebut wujud:

```tsx
<Link to="/users" />
```

Tetapi kod ini akan menghasilkan ralat TypeScript:

```tsx
<Link to="/nonsense" />
```

kerana `'/nonsense'` tidak terdapat dalam:

```ts
keyof FileRoutesByPath
```

Inilah sebab utama TanStack Router mempunyai type-safe navigation.

---

## Gambaran keseluruhan type pipeline

Keseluruhan proses boleh dilihat seperti ini:

```ts
createFileRoute('/')
```

### Langkah 1: Literal value

Nilai yang dihantar ialah:

```ts
'/'
```

TypeScript mengekalkannya sebagai literal type:

```ts
'/'
```

### Langkah 2: Generic inference

Generic function menerima:

```ts
TFilePath
```

TypeScript membuat inferens:

```ts
TFilePath = '/'
```

### Langkah 3: Generic return type

Return type menggunakan semula `TFilePath`:

```ts
FileRoute<TFilePath, ...>
```

Maka ia menjadi:

```ts
FileRoute<'/', ...>
```

### Langkah 4: Generated declaration

Fail `routeTree.gen.ts` memastikan `'/'` ialah key yang sah dalam:

```ts
FileRoutesByPath
```

### Langkah 5: Type tersebar ke seluruh router

Maklumat route tersebut kemudian digunakan oleh:

```ts
Route.fullPath
Route.useLoaderData()
Route.useParams()
Route.useSearch()
Link
navigate()
redirect()
```

Semua API tersebut mendapat type daripada route map yang sama.

---

## Aha moment

Keseluruhan type safety TanStack Router bermula daripada tiga perkara:

```text
Satu generic function
+
Satu literal value
+
Satu generated declaration
```

Iaitu:

```ts
createFileRoute('/')
```

bersama:

```ts
interface FileRoutesByPath {
  '/': ...
}
```

Daripada mekanisme ini, TypeScript boleh mengetahui path, params, loader data, search params dan navigation destination.

Semua hook, link dan pemeriksaan navigation dalam TanStack Router bergantung kepada type pipeline ini.

---

## 5. Buktikan dalam editor

Buka:

```text
src/routes/index.tsx
```

Kemudian hover pada:

```ts
Route
```

Editor seperti VS Code akan menunjukkan type yang lebih kurang seperti:

```ts
FileRoute<'/', ...>
```

Perhatikan bahawa path `'/'` disimpan sebagai literal type.

Kemudian cuba tulis:

```ts
Route.fullPath
```

Hover pada `fullPath`.

Type yang dipaparkan sepatutnya:

```ts
'/'
```

Bukan:

```ts
string
```

Anda juga boleh mengujinya seperti ini:

```ts
const path: '/' = Route.fullPath
```

Kod tersebut sepatutnya diterima.

Tetapi ini mungkin menghasilkan ralat:

```ts
const path: '/users' = Route.fullPath
```

kerana `Route.fullPath` ialah `'/'`, bukan `'/users'`.

---

Anda juga boleh menguji route yang tidak wujud:

```tsx
<Link to="/nonsense">Test</Link>
```

TypeScript sepatutnya menunjukkan red squiggle kerana path tersebut tidak terdapat dalam route tree.

Selepas itu, jalankan:

```bash
npm run dev
```

untuk memastikan aplikasi masih berjalan.

---

## 6. Kuiz ringkas

### Soalan 1

Apakah type untuk kod berikut?

```ts
let p = '/'
```

Pilihan jawapan:

* `'/'`
* `string`
* `any`
* `string[]`

Jawapan:

```text
string
```

Ini kerana variable `let` boleh ditukar kemudian, jadi TypeScript melakukan type widening.

---

### Soalan 2

Dalam kod berikut:

```ts
createFileRoute('/')
```

kenapa type argument tidak perlu ditulis?

Pilihan jawapan:

* Type argument inference
* Ia sebenarnya telah ditulis
* Type `any` digunakan
* Disebabkan `const`

Jawapan:

```text
Type argument inference
```

TypeScript membuat inferens `TFilePath` daripada nilai `'/'`.

---

### Soalan 3

Apakah yang dilakukan oleh `src/routeTree.gen.ts` apabila ia mengisytiharkan:

```ts
interface FileRoutesByPath
```

Pilihan jawapan:

* Mencipta route
* Menjalankan router
* Declaration merging
* Narrowing paths

Jawapan:

```text
Declaration merging
```

Ia menambah route sebenar kepada interface library melalui module augmentation.

---

### Soalan 4

Sekiranya aplikasi hanya mempunyai home route, apakah nilai:

```ts
keyof FileRoutesByPath
```

Pilihan jawapan:

* `string`
* `unknown`
* `any`
* `'/'`

Jawapan:

```ts
'/'
```

Ini kerana satu-satunya key dalam `FileRoutesByPath` ialah `'/'`.

---

## 7. Pergi lebih mendalam

Untuk memahami konsep ini dengan lebih kuat, pelajari topik berikut dalam TypeScript:

### Generics

Fokus kepada:

* Generic function
* Type parameter
* Generic constraints
* Type argument inference

Contoh asas:

```ts
function identity<T>(value: T): T {
  return value
}
```

### Literal types

Fokus kepada perbezaan antara:

```ts
const path = '/'
```

dan:

```ts
let path = '/'
```

Serta penggunaan:

```ts
as const
```

Contohnya:

```ts
let path = '/' as const
```

Walaupun menggunakan `let`, type akan dikekalkan sebagai:

```ts
'/'
```

### `keyof`

Operator `keyof` mengambil semua key daripada sesuatu object type.

Contoh:

```ts
interface Routes {
  '/': {}
  '/users': {}
}
```

Maka:

```ts
type RoutePath = keyof Routes
```

akan menghasilkan:

```ts
'/' | '/users'
```

### Declaration merging

TypeScript membenarkan beberapa declaration dengan nama interface yang sama digabungkan.

Contoh:

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

Hasil akhirnya dianggap seperti:

```ts
interface User {
  name: string
  email: string
}
```

TanStack Router menggunakan idea yang sama untuk menambah typed route map kepada interface library.

### Module augmentation

Module augmentation membolehkan anda menambah atau memperluas type daripada module lain.

Contoh:

```ts
declare module '@tanstack/react-router' {
  interface FileRoutesByPath {
    '/': {
      fullPath: '/'
    }
  }
}
```

---

# Ringkasan utama

Apabila anda menulis:

```ts
createFileRoute('/')
```

perkara berikut berlaku:

1. `'/'` dikekalkan sebagai literal type.
2. Generic function menangkap literal type tersebut sebagai `TFilePath`.
3. TypeScript mengisi generic type secara automatik melalui type inference.
4. `routeTree.gen.ts` memastikan `'/'` ialah route path yang sah.
5. Return type menyimpan path tersebut sebagai `FileRoute<'/', ...>`.
6. Type itu digunakan oleh `Link`, loader, params, search dan navigation.

Formula ringkasnya:

```text
Literal type
+ Generic function
+ Type inference
+ Generated route declaration
= Type-safe routing
```

Pelajaran seterusnya akan menerangkan proses loading data, termasuk bagaimana type daripada `loader` mengalir secara automatik ke dalam component melalui:

```ts
Route.useLoaderData()
```
