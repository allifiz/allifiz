Siap. Ini ulang dari awal untuk test lewat Superapps lokal port `4890`, tanpa shell env variable.

Semua request ke:

```text
http://localhost:4890/api/v1
```

Gunakan bearer token staging yang masih aktif bila local Superapps menjalankan guard. Bila `USE_GUARDS` lokal nonaktif, hapus header Authorization.

Data staging yang sudah valid:

```text
id_bundel existing      : 271254
id_soal_utama existing  : 3897522
kode_bab                : 06.00.01
id_soal_pengayaan       : 1
tahun_ajaran            : 2026/2027
id_klaster              : 2
id_level_kognitif       : 3
```

Untuk tiket 4–5, gunakan ID bundel baru hasil tiket 1 supaya tidak duplicate dengan data staging existing.

1. POST create bundel soal

```bash
curl --location 'http://localhost:4890/api/v1/smba/paket-bundel/bundel' \
  --header 'Authorization: Bearer <TOKEN_STAGING>' \
  --header 'Content-Type: application/json' \
  --data '{
    "deskripsi": "QA Pengayaan Local 20260730",
    "id_kelompok_ujian": 40,
    "id_tingkat_kelas": 41,
    "jumlah_soal": 1,
    "opsi_urut": "Nomor",
    "peruntukan": 88,
    "status": "Created",
    "tahun_ajaran": "2026/2027",
    "waktu_pengerjaan": 2,
    "updated_by": "2023182004",
    "semester": 1,
    "is_random": false,
    "is_skd": false,
    "is_pusat": true,
    "id_ptn": null,
    "jenis_bundel": "Ujian Sekolah",
    "id_kota": 7,
    "id_sekolah": 2918005,
    "id_sekre": 702,
    "has_soal_refinement": true,
    "has_soal_enrichment": false
  }'
```

Expected HTTP `201`:

```json
{
  "data": {
    "id": 271xxx,
    "kode_bundel": "2026/2027.40.41.xxxxx",
    "deskripsi": "QA Pengayaan Local 20260730"
  },
  "_meta": {
    "status": "success",
    "message": "success create bundle soal"
  }
}
```

Catat `data.id` hasilnya. Contoh di bawah saya tulis sebagai `<ID_BUNDEL_HASIL_STEP_1>`; ganti manual dengan angka hasil response.

2. PATCH update bundel soal

Test payload flag secara aman pada bundel staging existing `271254`. Nilainya sama dengan kondisi sekarang, jadi tidak mengubah status flag.

```bash
curl --location --request PATCH 'http://localhost:4890/api/v1/smba/paket-bundel/bundel/271254' \
  --header 'Authorization: Bearer <TOKEN_STAGING>' \
  --header 'Content-Type: application/json' \
  --data '{
    "id": 271254,
    "kode_bundel": "2026/2027.40.41.00016",
    "deskripsi": "Test Bundel TI2 - PATCH FLAGS",
    "waktu_pengerjaan": 2,
    "tahun_ajaran": "2026/2027",
    "jumlah_soal": 2,
    "jumlah_entri": 1,
    "status": "Created",
    "peruntukan": 88,
    "id_kelompok_ujian": 40,
    "id_tingkat_kelas": 41,
    "opsi_urut": "Nomor",
    "updated_by": "2023182004",
    "semester": 1,
    "id_sekolah": 2918005,
    "id_kota": 7,
    "id_sekre": 702,
    "is_pusat": true,
    "id_ptn": null,
    "jenis_bundel": "Ujian Sekolah",
    "is_random": false,
    "is_skd": false,
    "has_soal_refinement": false,
    "has_soal_enrichment": true
  }'
```

Expected: response sukses update. Validasi flag dilakukan pada tiket 3.

3. GET bundel soal

```bash
curl --location 'http://localhost:4890/api/v1/smba/paket-bundel/bundel?page=1&per_page=10&sort=desc&order_by=created_at&tahun_ajaran=2026%2F2027&searchBy=&jenis_bundel=Ujian%20Sekolah&keyword=&tingkat_kelas=41&peruntukan=88&kelompok_ujian=40' \
  --header 'Authorization: Bearer <TOKEN_STAGING>'
```

Expected: `data` berisi bundel `271254` dengan flag berikut.

```json
{
  "id": 271254,
  "has_soal_refinement": false,
  "has_soal_enrichment": true
}
```

Bundel hasil tiket 1 juga harus tampil dengan:

```json
{
  "has_soal_refinement": true,
  "has_soal_enrichment": false
}
```

4. POST penautan isi bundel soal

Ganti `<ID_BUNDEL_HASIL_STEP_1>` dengan ID numerik hasil tiket 1.

```bash
curl --location 'http://localhost:4890/api/v1/smba/paket-bundel/bundel/isi' \
  --header 'Authorization: Bearer <TOKEN_STAGING>' \
  --header 'Content-Type: application/json' \
  --data '{
    "id_soal": [3897522],
    "id_bundel": <ID_BUNDEL_HASIL_STEP_1>,
    "kode_bab": "06.00.01",
    "updated_by": "2023182004",
    "soal_refinement_ids": [1],
    "soal_enrichment_ids": [1]
  }'
```

Expected HTTP `201`:

```json
{
  "data": 1,
  "_meta": {
    "status": "success",
    "message": "success add many isi bundle soal"
  }
}
```

`data: 1` berarti satu parent question berhasil ditautkan ke bundel.

5. GET isi bundel soal

Gunakan ID bundel dari tiket 1.

```bash
curl --location 'http://localhost:4890/api/v1/smba/paket-bundel/bundel/<ID_BUNDEL_HASIL_STEP_1>/isi?is_all_data=true&order_by=nomor_soal&sort=asc&c_id_soal=3897522' \
  --header 'Authorization: Bearer <TOKEN_STAGING>'
```

Expected item:

```json
{
  "id_soal": 3897522,
  "kode_bab": "06.00.01",
  "soal_refinement_ids": [1],
  "soal_enrichment_ids": [1]
}
```

6. GET soal pengayaan

Ini memakai data staging existing, sehingga bisa langsung dijalankan tanpa create data baru.

```bash
curl --location 'http://localhost:4890/api/v1/smba/soal-pengayaan?page=1&per_page=10&tahun_ajaran=2026%2F2027&is_refinement=true&is_enrichment=true&id_soal_utama=3897522' \
  --header 'Authorization: Bearer <TOKEN_STAGING>'
```

Expected:

```json
{
  "data": [
    {
      "id_soal": 1,
      "cuplikan_soal": "<HTML soal pengayaan>",
      "jenis": ["REFINEMENT", "ENRICHMENT"]
    }
  ],
  "metadata": {
    "total_count": 1,
    "page": 1,
    "per_page": 10
  },
  "_meta": {
    "code": 200,
    "status": "success",
    "message": "success get soal pengayaan"
  }
}
```

7. GET isi bundel soal pengayaan

```bash
curl --location 'http://localhost:4890/api/v1/smba/soal-pengayaan/isi-bundel?id_bundel=271254&id_soal_utama=3897522' \
  --header 'Authorization: Bearer <TOKEN_STAGING>'
```

Expected:

```json
{
  "data": [
    {
      "id_soal": 1,
      "cuplikan_soal": "<HTML soal pengayaan>",
      "jenis": ["REFINEMENT", "ENRICHMENT"]
    }
  ],
  "_meta": {
    "code": 200,
    "status": "success",
    "message": "success get isi bundel soal pengayaan"
  }
}
```

8. PATCH update isi bundel soal pengayaan

Payload ini idempotent terhadap data staging saat ini: hanya menyimpan ulang array yang sama.

```bash
curl --location --request PATCH 'http://localhost:4890/api/v1/smba/soal-pengayaan/isi-bundel' \
  --header 'Authorization: Bearer <TOKEN_STAGING>' \
  --header 'Content-Type: application/json' \
  --data '{
    "id_bundel": 271254,
    "id_soal_utama": 3897522,
    "soal_refinement_ids": [1],
    "soal_enrichment_ids": [1]
  }'
```

Expected:

```json
{
  "data": true,
  "_meta": {
    "code": 200,
    "status": "success",
    "message": "success update isi bundel soal pengayaan"
  }
}
```

Sesudahnya ulangi tiket 7. `jenis` harus tetap berisi `REFINEMENT` dan `ENRICHMENT`.

9. POST create soal pengayaan

`opsi` wajib dikirim sebagai string JSON, bukan object JSON biasa.

```bash
curl --location --request POST 'http://localhost:4890/api/v1/smba/soal-pengayaan' \
  --header 'Authorization: Bearer <TOKEN_STAGING>' \
  --header 'Content-Type: application/json' \
  --data '{
    "tahun_ajaran": "2026/2027",
    "semester": 1,
    "soal": "<p>QA soal pengayaan local 20260730</p>",
    "tingkat_kesulitan": "MUDAH",
    "tipe_soal": "PGB",
    "opsi": "{\"opsi\":{\"A\":\"Jawaban A\",\"B\":\"Jawaban B\"},\"jawaban\":\"A\"}",
    "jenjang": "SD",
    "id_tingkat_kelas": 3,
    "id_klaster": 2,
    "id_level_kognitif": 3,
    "is_refinement": true,
    "is_enrichment": false,
    "created_by": "2023182004",
    "solusi": "<p>Solusi QA</p>",
    "the_king": "A"
  }'
```

Expected body:

```json
{
  "data": true,
  "_meta": {
    "code": 200,
    "status": "success",
    "message": "success create soal pengayaan"
  }
}
```

Untuk Postman: pakai method, URL, header Authorization, dan body yang persis sama seperti curl di atas. Tidak perlu environment variable; cukup paste URL `localhost:4890` dan ganti manual ID bundel hasil tiket 1 pada tiket 4–5.
