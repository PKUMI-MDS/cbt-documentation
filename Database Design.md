\# Database Design - CBT Laravel



\## Prinsip Desain



Desain database ini disusun untuk mendukung model CBT berbasis sesi dengan aturan berikut:



\- Admin dapat membuat sesi tes yang memiliki tanggal dan waktu pelaksanaan.

\- Admin dapat menentukan apakah nilai dapat dilihat oleh user atau tidak per sesi.

\- Admin dapat menentukan batas jumlah peserta pada setiap sesi.

\- Sistem dapat melakukan auto generate user yang sudah membayar dan sudah di-approve admin ke dalam sesi.

\- Durasi pengerjaan satu sesi adalah 2 jam atau 120 menit.

\- Satu kali approval tes setelah upload bukti pembayaran hanya berlaku untuk satu kali tes.

\- Jika user ingin tes lagi, user harus upload bukti pembayaran baru dan mendapatkan approval baru untuk sesi yang berbeda.



\## Daftar Tabel Inti



\### users

\- id

\- name

\- email

\- password

\- role

\- account\_status

\- approval\_at

\- approved\_by

\- last\_login\_at

\- created\_at

\- updated\_at



\### user\_profiles

\- id

\- user\_id

\- phone

\- institution

\- identity\_number

\- address

\- created\_at

\- updated\_at



\### payment\_proofs

Menyimpan semua riwayat upload bukti pembayaran user.



\- id

\- user\_id

\- file\_path

\- file\_name

\- mime\_type

\- amount

\- payment\_date

\- status

\- reviewed\_by

\- reviewed\_at

\- rejection\_reason

\- created\_at

\- updated\_at



\### test\_approvals

Mewakili hak ikut tes yang lahir dari satu bukti pembayaran yang disetujui admin.



\- id

\- user\_id

\- payment\_proof\_id

\- approval\_code

\- approved\_by

\- approved\_at

\- quota\_total

\- quota\_used

\- status

\- notes

\- created\_at

\- updated\_at



\### question\_banks

\- id

\- code

\- title

\- description

\- is\_active

\- created\_by

\- created\_at

\- updated\_at



\### questions

\- id

\- question\_bank\_id

\- section\_type

\- question\_type

\- difficulty\_level

\- stem\_html

\- explanation\_html

\- image\_path

\- audio\_path

\- audio\_max\_play\_count

\- is\_active

\- created\_by

\- updated\_by

\- created\_at

\- updated\_at



\### question\_options

\- id

\- question\_id

\- option\_key

\- option\_html

\- is\_correct

\- sort\_order

\- created\_at

\- updated\_at



\### exam\_packages

Paket ujian adalah blueprint ujian yang nanti dipakai oleh sesi.



\- id

\- code

\- title

\- description

\- duration\_minutes

\- shuffle\_questions

\- shuffle\_options

\- max\_tab\_switch

\- max\_fullscreen\_exit

\- is\_active

\- created\_at

\- updated\_at



\### exam\_package\_banks

\- id

\- exam\_package\_id

\- question\_bank\_id

\- section\_type

\- question\_count

\- sort\_order

\- created\_at

\- updated\_at



\### exam\_sessions

Sesi tes master yang dibuat admin.



\- id

\- exam\_package\_id

\- code

\- title

\- description

\- session\_date

\- start\_time

\- end\_time

\- duration\_minutes

\- max\_participants

\- show\_result\_to\_user

\- auto\_generate\_enabled

\- auto\_generate\_mode

\- access\_code

\- status

\- created\_by

\- created\_at

\- updated\_at



\### exam\_session\_registrations

Relasi peserta ke sesi tes tertentu.



\- id

\- exam\_session\_id

\- user\_id

\- test\_approval\_id

\- assignment\_type

\- registration\_status

\- assigned\_by

\- assigned\_at

\- notes

\- created\_at

\- updated\_at



\### exam\_attempts

Runtime ujian saat user benar-benar mulai mengerjakan sesi.



\- id

\- exam\_session\_registration\_id

\- user\_id

\- exam\_session\_id

\- exam\_package\_id

\- status

\- started\_at

\- ends\_at

\- submitted\_at

\- last\_activity\_at

\- remaining\_seconds

\- current\_question\_number

\- total\_questions

\- snapshot\_version

\- session\_token

\- ip\_address

\- user\_agent

\- created\_at

\- updated\_at



\### exam\_attempt\_questions

Snapshot urutan soal per user pada attempt tertentu.



\- id

\- exam\_attempt\_id

\- question\_id

\- display\_number

\- option\_order\_json

\- section\_type

\- is\_doubtful

\- question\_snapshot\_json

\- created\_at

\- updated\_at



\### exam\_answers

\- id

\- exam\_attempt\_id

\- exam\_attempt\_question\_id

\- selected\_option\_id

\- answered\_at

\- is\_correct

\- score\_value

\- created\_at

\- updated\_at



\### exam\_results

\- id

\- exam\_attempt\_id

\- exam\_session\_id

\- user\_id

\- total\_score

\- listening\_score

\- structure\_score

\- reading\_score

\- correct\_count

\- wrong\_count

\- unanswered\_count

\- published\_at

\- created\_at

\- updated\_at



\### violation\_logs

\- id

\- exam\_attempt\_id

\- violation\_type

\- severity

\- event\_payload\_json

\- action\_taken

\- created\_at

\- updated\_at



\### audio\_play\_logs

\- id

\- exam\_attempt\_id

\- question\_id

\- play\_count

\- last\_played\_at

\- created\_at

\- updated\_at



\### user\_sessions

\- id

\- user\_id

\- login\_token

\- device\_name

\- ip\_address

\- user\_agent

\- is\_active

\- last\_seen\_at

\- created\_at

\- updated\_at



\### settings

\- id

\- group\_key

\- item\_key

\- item\_value

\- created\_at

\- updated\_at



\### activity\_logs

\- id

\- actor\_id

\- subject\_type

\- subject\_id

\- action

\- properties\_json

\- created\_at

\- updated\_at



\## Relasi Utama



\- users hasOne user\_profiles

\- users hasMany payment\_proofs

\- users hasMany test\_approvals

\- users hasMany exam\_session\_registrations

\- users hasMany exam\_attempts

\- payment\_proofs belongsTo users

\- payment\_proofs hasOne test\_approvals

\- test\_approvals belongsTo users

\- test\_approvals belongsTo payment\_proofs

\- question\_banks hasMany questions

\- questions hasMany question\_options

\- exam\_packages belongsToMany question\_banks melalui exam\_package\_banks

\- exam\_sessions belongsTo exam\_packages

\- exam\_sessions hasMany exam\_session\_registrations

\- exam\_session\_registrations belongsTo exam\_sessions

\- exam\_session\_registrations belongsTo users

\- exam\_session\_registrations belongsTo test\_approvals

\- exam\_session\_registrations hasOne exam\_attempts

\- exam\_attempts belongsTo exam\_session\_registrations

\- exam\_attempts hasMany exam\_attempt\_questions

\- exam\_attempts hasMany exam\_answers

\- exam\_attempts hasOne exam\_results

\- exam\_attempts hasMany violation\_logs



\## Enum yang Disarankan



\### account\_status

\- pending\_verification

\- active

\- rejected

\- suspended



\### payment\_proof\_status

\- pending\_review

\- approved

\- rejected



\### test\_approval\_status

\- available

\- consumed

\- cancelled

\- expired



\### section\_type

\- listening

\- structure

\- reading

\- arabic\_grammar

\- arabic\_reading



\### question\_type

\- multiple\_choice\_text

\- multiple\_choice\_image

\- multiple\_choice\_audio

\- multiple\_choice\_mixed



\### exam\_session\_status

\- draft

\- published

\- closed

\- finished

\- cancelled



\### exam\_session\_registration\_status

\- assigned

\- confirmed

\- cancelled

\- completed

\- absent



\### exam\_attempt\_status

\- not\_started

\- in\_progress

\- submitted

\- expired

\- auto\_submitted

\- cancelled



\### assignment\_type

\- manual

\- auto



\### violation\_type

\- tab\_switch

\- fullscreen\_exit

\- right\_click

\- copy\_attempt

\- multi\_login\_attempt

\- suspicious\_disconnect



\## Aturan Bisnis di Level Database



\### Approval Tes

\- Satu `payment\_proof` yang di-approve menghasilkan satu `test\_approval`.

\- Nilai default `quota\_total = 1`.

\- Nilai default `quota\_used = 0`.

\- Jika approval dipakai untuk ikut satu sesi dan attempt dibuat, approval dianggap terpakai.

\- Satu `test\_approval` tidak boleh dipakai untuk lebih dari satu tes.



\### Sesi Tes

\- Satu `exam\_session` memiliki tanggal tes, jam mulai, jam selesai, dan `duration\_minutes = 120`.

\- Satu `exam\_session` memiliki `max\_participants`.

\- Satu `exam\_session` memiliki `show\_result\_to\_user` untuk menentukan apakah user boleh melihat nilai.



\### Assignment Peserta

\- Satu user hanya bisa masuk ke sesi jika memiliki `test\_approval` dengan status `available`.

\- Satu `test\_approval` hanya boleh direlasikan ke satu `exam\_session\_registration`.

\- Assignment otomatis hanya boleh mengambil user yang sudah memiliki pembayaran approved dan approval tes yang masih tersedia.

\- Jika kuota sesi penuh, registrasi baru harus ditolak.



\### Retake

\- User yang ingin tes lagi harus membuat `payment\_proof` baru.

\- Jika payment baru di-approve, sistem membuat `test\_approval` baru.

\- `test\_approval` baru harus dipakai untuk sesi yang berbeda dari tes sebelumnya.



\## Catatan Desain Penting



\### Kenapa perlu test\_approvals

Tabel ini penting untuk memisahkan antara status akun dan hak ikut tes. User bisa tetap memiliki akun aktif, tetapi hak tes tetap dihitung per pembayaran yang di-approve.



\### Kenapa perlu exam\_session\_registrations

Tabel ini penting untuk memisahkan master sesi dengan peserta yang masuk ke sesi. Dengan pendekatan ini, kuota sesi, auto generate peserta, dan assignment manual dapat dikelola lebih rapi.



\### Kenapa perlu exam\_attempts

Satu user belum tentu langsung mengerjakan ujian meskipun sudah dimasukkan ke sesi. Karena itu runtime ujian perlu dipisah dari tabel registrasi sesi.



\### Kenapa perlu exam\_attempt\_questions

Tabel ini penting untuk menyimpan snapshot urutan soal per user. Tanpa tabel ini, randomisasi akan sulit dijaga konsistensinya saat refresh atau resume.



\### Kenapa perlu option\_order\_json

Meskipun opsi berasal dari `question\_options`, urutan tampil per attempt bisa berbeda. Karena itu urutan perlu disimpan di level snapshot.



\### Kenapa hasil dipisah di exam\_results

Perhitungan skor lebih rapi, mudah di-export, dan tidak perlu menghitung ulang dari nol setiap kali dashboard dibuka.



\## Constraint yang Disarankan



\- Unique: `users.email`

\- Unique: `test\_approvals.payment\_proof\_id`

\- Unique: `exam\_session\_registrations.test\_approval\_id`

\- Check: `exam\_sessions.duration\_minutes = 120`

\- Check: `exam\_sessions.max\_participants > 0`

\- Check: `test\_approvals.quota\_total = 1`

\- Check: `test\_approvals.quota\_used IN (0,1)`



\## Index yang Disarankan



\- users(email)

\- users(account\_status)

\- payment\_proofs(user\_id, status)

\- test\_approvals(user\_id, status)

\- test\_approvals(payment\_proof\_id)

\- questions(question\_bank\_id, section\_type, is\_active)

\- exam\_sessions(session\_date, start\_time, status)

\- exam\_sessions(exam\_package\_id, status)

\- exam\_session\_registrations(exam\_session\_id, registration\_status)

\- exam\_session\_registrations(user\_id, exam\_session\_id)

\- exam\_attempts(user\_id, status)

\- exam\_attempt\_questions(exam\_attempt\_id, display\_number)

\- exam\_answers(exam\_attempt\_id, exam\_attempt\_question\_id)

\- violation\_logs(exam\_attempt\_id, violation\_type)

