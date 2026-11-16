# rclone bisync: শুরু থেকে টিউটোরিয়াল (শুধু CMD দিয়ে)

নিচের ধাপগুলো ক্রমানুসারে করুন। প্রতিটি কমান্ড CMD-তে কপি-পেস্ট করলেই চলবে।

## ধাপ ১: CMD খুলুন ও ড্রাইভে যান

```
D:
cd D:\Drive
```

## ধাপ ২: rclone ঠিকমতো আছে কিনা দেখুন

```
rclone version
```

`bisync`-এর জন্য v1.58 বা তার উপরের ভার্সন লাগে (আপনার v1.76.0, ঠিক আছে)।

## ধাপ ৩: remote-গুলো দেখুন

```
rclone listremotes
```

আপনার remote: `Drive-zahid2.kwsr:`

## ধাপ ৪: Drive-এর ফোল্ডার দেখে নিন

```
rclone lsd "Drive-zahid2.kwsr:"
rclone lsd "Drive-zahid2.kwsr:Kwsr1"
rclone lsd "Drive-zahid2.kwsr:Kwsr1/Documents/DPDC"
```

এভাবে এক লেভেল করে ভেতরে ঢুকে সঠিক পাথ নিশ্চিত করুন।

## ধাপ ৫: লোকাল ফোল্ডার বানান

```
mkdir D:\Drive\DPDC
```

এটা আগে না বানালে "directory not found" error আসে।

## ধাপ ৬: ড্রাই-রান (কিছুই বদলাবে না, শুধু দেখাবে)

```
rclone bisync "Drive-zahid2.kwsr:Kwsr1/Documents/DPDC" "D:\Drive\DPDC" --resync --create-empty-src-dirs --drive-skip-gdocs --max-delete 20 --dry-run -P
```

- `--resync`: প্রথমবার বাধ্যতামূলক, দুই পাশ মিলিয়ে শুরুর অবস্থা (baseline) বানায়।
- `--create-empty-src-dirs`: খালি ফোল্ডারও সিঙ্ক হবে।
- `--drive-skip-gdocs`: Google Docs/Sheets-এর নিজস্ব ফরম্যাট বাদ দেবে।
- `--max-delete 20`: একবারে ২০ শতাংশের বেশি ফাইল মোছা ঠেকাবে।
- `--dry-run`: আসলে কিছু করবে না।
- `-P`: প্রগ্রেস দেখাবে।

আউটপুটে error না থাকলে পরের ধাপে যান।

## ধাপ ৭: প্রথম আসল রান

একই কমান্ড, শুধু `--dry-run` ছাড়া:

```
rclone bisync "Drive-zahid2.kwsr:Kwsr1/Documents/DPDC" "D:\Drive\DPDC" --resync --create-empty-src-dirs --drive-skip-gdocs --max-delete 20 -P
```

Drive-এর সব ফাইল `D:\Drive\DPDC`-এ নামবে। `Z-Deleted` ফোল্ডারও সিঙ্ক হবে, কারণ কোনো `--exclude` দেওয়া নেই।

## ধাপ ৮: রোজকার সিঙ্ক

এখন থেকে `--resync` ছাড়া:

```
rclone bisync "Drive-zahid2.kwsr:Kwsr1/Documents/DPDC" "D:\Drive\DPDC" --create-empty-src-dirs --drive-skip-gdocs --max-delete 20 -P
```

কাজ শেষে একবার চালালেই লোকাল ও Drive-এর পরিবর্তন দুই দিকেই মিলে যাবে।

## ধাপ ৯: কখন `--resync` আবার লাগবে

- bisync "must run --resync to recover" বললে।
- অপশন বদলালে (যেমন `--exclude` যোগ করলে)।
- পরপর অনেকদিন সিঙ্ক বন্ধ থাকলে ও error এলে।

তখন ধাপ ৭-এর কমান্ড আবার চালান। এতে ফাইল মুছে না, শুধু baseline নতুন করে বানায়।

## গুরুত্বপূর্ণ নিয়ম

- প্রতিবার একই অপশন দিয়ে চালান, নইলে bisync আপত্তি করবে।
- এক পাশে ফাইল মুছলে বা নাম বদলালে পরের সিঙ্কে অন্য পাশেও তা হবে।
- দুই পাশে একই ফাইল বদলালে একটার নাম বদলে `.conflict1` হয়ে দুটোই থাকে।
- সিঙ্ক চলার সময় ফাইল খোলা বা বদলানো ঠিক না।
- আগের সিঙ্ক শেষ না হতে আরেকটা চালাবেন না, নইলে lock error আসে।
- অডিটের ফাইল বলে মাঝে মাঝে `D:\Drive\DPDC`-এর ব্যাকআপ রাখা নিরাপদ।

## সমস্যা হলে

| সমস্যা | সমাধান |
|---|---|
| `directory not found` | `mkdir D:\Drive\DPDC` দিয়ে লোকাল ফোল্ডার বানান |
| `must run --resync` | ধাপ ৭-এর কমান্ড চালান |
| `prior lock file found` | আরেকটা সিঙ্ক চলছে কিনা দেখুন, না থাকলে `%USERPROFILE%\AppData\Local\rclone\bisync` ফোল্ডারের `.lck` ফাইল মুছুন |
| `too many deletes` | আসলেই মোছা দরকার কিনা দেখে নিন, দরকার হলে `--force` যোগ করুন |

কোনো ধাপে error এলে আউটপুট পেস্ট করুন।
