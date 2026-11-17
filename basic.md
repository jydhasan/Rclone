## সূচিপত্র
[🏠 হোম](README.md) | [☁️ rclone](rclone.md) | [🔧 সমস্যা সমাধান](SimpleCMD.md) | [Basic CMD](basic.md)

হ্যাঁ, যায়। এটাই সবচেয়ে ভালো সিকিউরিটি। এটাকে বলে `--max-delete`

এর মানে হলো, একবারে এর বেশি ফাইল ডিলিট হতে দিবে না। ভুল করে বেশি ফাইল ডিলিট হতে গেলেই রক্লোন থেমে যাবে।

### ১. Sync এর জন্য:

```bash
rclone sync gdrive:backup "D:\Backup" --max-delete 10 -P
```

মানে একবারে ১০টার বেশি ফাইল ডিলিট হবে না। ১১ নাম্বার ফাইল ডিলিট হতে গেলেই সিঙ্ক বন্ধ হয়ে যাবে।

সাইজ দিয়েও আটকাতে পারবা:
```bash
rclone sync gdrive:backup "D:\Backup" --max-delete 10 --max-delete-size 100M -P
```
মানে ১০টা ফাইল বা ১০০ MB এর বেশি ডিলিট হবে না।

### ২. Bisync এর জন্য (তোমারটা):

```bash
rclone bisync gdrive:backup "D:\Backup" --max-delete 10 -P
```

প্রথমবার `--resync` এর সাথেও দিতে পারবা:
```bash
rclone bisync gdrive:backup "D:\Backup" --resync --max-delete 10 -P
```

### আরো ১টা সিকিউরিটি - ডিলিট না করে ব্যাকআপ ফোল্ডারে রাখা:

ডিলিট না করে আলাদা ফোল্ডারে সরিয়ে রাখতে চাইলে:

```bash
rclone sync gdrive:backup "D:\Backup" --backup-dir gdrive:trash-backup -P
```

এতে ভুল করে ডিলিট হলেও ফাইলটা `trash-backup` ফোল্ডারে থেকে যাবে, পুরো হারাবে না।

তুমি কয়টা ফাইল পর্যন্ত ডিলিট অ্যালাউ করতে চাও? ১০টা দিবো নাকি ৫টা? বললে তোমার ফাইনাল সিকিউর কমান্ডটা বানিয়ে দেই।
