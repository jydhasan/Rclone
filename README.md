## সূচিপত্র
[🏠 হোম](README.md) | [☁️ rclone](rclone.md) | [🔧 সমস্যা সমাধান](troubleshooting.md)
# Windows এ rclone দিয়ে Google Drive সেটআপ গাইড

এই গাইড অনুসরণ করে যেকোনো কম্পিউটারে বা যেকোনো Google অ্যাকাউন্টে rclone দিয়ে Google Drive কানেক্ট করা যাবে।

> **সতর্কতা:** `client_secret`, `rclone.conf` ফাইল, বা কোনো token এই repo তে **কখনো commit করবেন না**। নিচে `.gitignore` এর নির্দেশনা আছে।

---

[Claude preview](https://claude.ai/share/ae137aae-6a4f-4145-8636-208770a2af86)
## সূচিপত্র

1. [rclone ইনস্টল](#১-rclone-ইনস্টল)
2. [Google Cloud এ নিজের OAuth Client তৈরি](#২-google-cloud-এ-নিজের-oauth-client-তৈরি)
3. [rclone এ Google Drive রিমোট কনফিগার](#৩-rclone-এ-google-drive-রিমোট-কনফিগার)
4. [টেস্ট](#৪-টেস্ট)
5. [দৈনন্দিন কমান্ড](#৫-দৈনন্দিন-কমান্ড)
6. [অন্য কম্পিউটার বা অন্য অ্যাকাউন্টে সেটআপ](#৬-অন্য-কম্পিউটার-বা-অন্য-অ্যাকাউন্টে-সেটআপ)
7. [সমস্যা সমাধান](#৭-সমস্যা-সমাধান)
8. [নিরাপত্তা ও .gitignore](#৮-নিরাপত্তা-ও-gitignore)

---

## ১. rclone ইনস্টল

### অপশন A: winget (সহজ)

```powershell
winget install Rclone.Rclone
```

ইনস্টলের পর **PowerShell পুরোপুরি বন্ধ করে নতুন করে খুলুন**, কারণ PATH আপডেট নতুন উইন্ডোতে কার্যকর হয়।

যাচাই করুন:

```powershell
rclone version
```

### অপশন B: ম্যানুয়াল

1. https://rclone.org/downloads/ থেকে Windows zip ডাউনলোড করুন
2. `C:\rclone` এ extract করুন
3. Start → "Edit the system environment variables" → Environment Variables → Path → New → `C:\rclone`
4. নতুন PowerShell খুলে `rclone version` চালান

### যদি `rclone` কমান্ড না পাওয়া যায়

নতুন উইন্ডোতেও না পেলে বর্তমান সেশনে PATH রিফ্রেশ করুন:

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
rclone version
```

অথবা exe খুঁজে বের করুন:

```powershell
Get-ChildItem "$env:LOCALAPPDATA\Microsoft\WinGet\Packages" -Recurse -Filter rclone.exe | Select-Object FullName
```

> পরামর্শ: Admin PowerShell (`C:\Windows\system32`) এর বদলে সাধারণ PowerShell ব্যবহার করুন। Admin শেলে config অন্য ইউজার প্রোফাইলে সেভ হতে পারে।

---

## ২. Google Cloud এ নিজের OAuth Client তৈরি

rclone এর শেয়ার্ড client_id ২০২৬ সালের মধ্যে বন্ধ হয়ে যাবে বলে ঘোষণা আছে, তাই নিজের client_id বানানো ভালো।

### ২.১ প্রজেক্ট তৈরি

1. https://console.cloud.google.com এ লগইন করুন
2. উপরের প্রজেক্ট ড্রপডাউন → **New Project** → নাম `rclone` → **Create**
3. নতুন প্রজেক্টটি সিলেক্ট আছে কিনা নিশ্চিত করুন

### ২.২ Google Drive API চালু করুন

1. উপরের সার্চ বারে `Google Drive API` লিখুন
2. **Enable** চাপুন (আগে চালু থাকলে "Manage" দেখাবে)

### ২.৩ Google Auth Platform (Consent screen) সেটআপ

**Google Auth Platform → Get started** (বা Overview) এ গিয়ে ৪টি ধাপ শেষ করুন:

| ধাপ | কী করবেন |
|---|---|
| App Information | App name: `rclone`, User support email: নিজের ইমেইল |
| Audience | **External** |
| Contact Information | নিজের ইমেইল |
| Finish | Policy checkbox টিক দিয়ে **Continue → Create** |

Branding পেজে logo, homepage, privacy policy ইত্যাদি **দিতে হবে না**। Logo দিলে অ্যাপ verification এ জমা দিতে হয়, তাই এড়িয়ে চলুন।

### ২.৪ OAuth Client তৈরি

1. বাম মেনু → **Clients** → **Create client**
2. Application type: **Desktop app**
3. Name: `rclone`
4. **"Use this client for an AI-powered agent"** চেকবক্স থাকলে আনচেক করুন
5. **Create** চাপুন
6. **Client ID** আর **Client secret** কপি করে **নিরাপদ জায়গায়** (password manager) রাখুন। এই repo তে রাখবেন না।

> নতুন client চালু হতে ৫ মিনিট থেকে কয়েক ঘণ্টা লাগতে পারে।

### ২.৫ Test user যোগ করুন (অথবা Publish)

Testing মোডে শুধু test user রা লগইন করতে পারে। ব্যক্তিগত ব্যবহারে এটাই সহজ:

1. বাম মেনু → **Audience**
2. **Test users** → **Add users**
3. যে Gmail দিয়ে Drive ব্যবহার করবেন সেটি যোগ করে **Save**

**সীমাবদ্ধতা:** Testing মোডে refresh token **৭ দিন পর মেয়াদ হারায়**। তখন শুধু এই কমান্ড চালাতে হয়:

```powershell
rclone config reconnect gdrive:
```

**বিকল্প:** Audience পেজে **Publish app** করলে (In production) ৭ দিনের সমস্যা থাকে না। ব্রাউজারে "Google hasn't verified this app" এলে **Advanced → Go to rclone (unsafe)** চাপুন। নিজের অ্যাপ হওয়ায় সমস্যা নেই।

---

## ৩. rclone এ Google Drive রিমোট কনফিগার

```powershell
rclone config
```

প্রম্পট অনুযায়ী উত্তর:

| প্রম্পট | কী লিখবেন |
|---|---|
| `e/n/d/r/c/s/q>` | `n` (new remote) |
| `name>` | `gdrive` |
| `Storage>` | `24` অথবা `drive` (তালিকার নম্বর বদলাতে পারে, তাই `drive` লেখা নিরাপদ) |
| `client_id>` | Google Cloud থেকে পাওয়া **Client ID** পেস্ট করুন |
| `client_secret>` | **Client secret** পেস্ট করুন |
| `scope>` | `1` (Full access) |
| `service_account_file>` | শুধু **Enter** |
| `Edit advanced config?` | `n` |
| `Use web browser to automatically authenticate?` | `y` |
| (ব্রাউজার খুলবে) | Google অ্যাকাউন্টে লগইন করে **Allow** দিন |
| `Configure this as a Shared Drive (Team Drive)?` | `n` (Shared Drive হলে `y`) |
| `Keep this "gdrive" remote?` | `y` |
| `e/n/d/r/c/s/q>` | `q` |

### ব্রাউজারে যা ঘটতে পারে

| বার্তা | সমাধান |
|---|---|
| "Google hasn't verified this app" | **Advanced → Go to rclone (unsafe)** → Allow |
| `Access blocked ... Error 403: access_denied` | Audience → Test users এ নিজের Gmail যোগ করুন, তারপর আবার চেষ্টা করুন |
| `invalid_client` | ৫-১০ মিনিট অপেক্ষা করে আবার চেষ্টা করুন |
| ব্রাউজার খুললই না | টার্মিনালে দেখানো `http://127.0.0.1:53682/auth?state=...` লিংকটি কপি করে ব্রাউজারে খুলুন |

---

## ৪. টেস্ট

```powershell
rclone listremotes        # gdrive: দেখাবে
rclone lsd gdrive:        # Drive এর টপ-লেভেল ফোল্ডার দেখাবে
```

ফোল্ডারের তালিকা এলে সেটআপ সম্পূর্ণ।

---

## ৫. দৈনন্দিন কমান্ড

### দেখা

```powershell
rclone lsd gdrive:                       # শুধু ফোল্ডার (এক লেভেল)
rclone lsd gdrive:MSC -R                 # সাবফোল্ডারসহ সব ফোল্ডার
rclone ls gdrive:MSC                     # ফাইল, সাইজসহ (সব সাবফোল্ডার ধরে)
rclone lsl gdrive:MSC                    # ফাইল, সাইজ ও তারিখসহ
rclone lsf gdrive:MSC --dirs-only        # শুধু ফোল্ডারের নাম
rclone lsf gdrive:MSC --files-only       # শুধু ফাইলের নাম
rclone tree gdrive:MSC                   # গাছের মতো কাঠামো
```

| কমান্ড | কী দেখায় |
|---|---|
| `lsd` | শুধু ফোল্ডার (এক লেভেল) |
| `ls` | শুধু ফাইল, সাইজসহ |
| `lsl` | ফাইল, সাইজ ও তারিখসহ |
| `lsf` | সহজ নামের তালিকা |
| `tree` | পুরো কাঠামো গাছের মতো |

ফোল্ডারের নামে স্পেস থাকলে কোটেশন দিন:

```powershell
rclone lsd "gdrive:My Folder/Sub Folder"
```

### কপি ও সিঙ্ক

```powershell
rclone copy C:\data gdrive:backup --progress      # লোকাল থেকে Drive এ (কিছু মোছে না)
rclone copy gdrive:backup C:\restore --progress   # Drive থেকে লোকালে
rclone sync C:\data gdrive:backup --dry-run       # sync এর প্রিভিউ
rclone sync C:\data gdrive:backup --progress      # গন্তব্যকে সোর্সের মতো বানায়
```

> **`sync` সাবধানে:** গন্তব্য থেকে ফাইল **মুছে ফেলতে পারে**। প্রথমবার সবসময় `--dry-run` দিন। শুধু যোগ করতে চাইলে `copy` ব্যবহার করুন।

### মাউন্ট (ঐচ্ছিক)

Drive কে একটি ড্রাইভ লেটার হিসেবে দেখতে চাইলে প্রথমে WinFsp লাগবে:

```powershell
winget install WinFsp.WinFsp
rclone mount gdrive: X: --vfs-cache-mode full
```

কমান্ড চলতে থাকা অবস্থায় `X:` ড্রাইভ হিসেবে দেখা যাবে।

---

## ৬. অন্য কম্পিউটার বা অন্য অ্যাকাউন্টে সেটআপ

### A. একই Google অ্যাকাউন্ট, নতুন কম্পিউটার

দুটি উপায়:

**উপায় ১: নতুন করে কনফিগার (সুপারিশকৃত)**

1. [ধাপ ১](#১-rclone-ইনস্টল) অনুযায়ী rclone ইনস্টল করুন
2. [ধাপ ৩](#৩-rclone-এ-google-drive-রিমোট-কনফিগার) অনুসরণ করুন। **একই Client ID ও Client secret** আবার ব্যবহার করা যাবে, নতুন client বানাতে হবে না
3. ব্রাউজারে লগইন করুন

**উপায় ২: config ফাইল কপি**

পুরোনো কম্পিউটারে config ফাইলের অবস্থান দেখুন:

```powershell
rclone config file
```

সাধারণত: `%APPDATA%\rclone\rclone.conf`

এই ফাইলটি **নিরাপদ মাধ্যমে** (USB, এনক্রিপ্টেড ড্রাইভ) নতুন কম্পিউটারে একই পাথে রাখুন। এতে token ও secret আছে, তাই ইমেইল বা পাবলিক জায়গায় পাঠাবেন না।

### B. অন্য Google অ্যাকাউন্ট

1. Google Cloud Console এর **Audience → Test users** এ নতুন Gmail যোগ করুন (একই client ব্যবহার করলে)
2. নতুন রিমোটের জন্য আলাদা নাম দিন:

```powershell
rclone config
# n → name: gdrive2 → drive → একই client_id/secret → ... → নতুন অ্যাকাউন্ট দিয়ে লগইন
```

3. ব্যবহার:

```powershell
rclone lsd gdrive2:
```

> প্রতিটি অ্যাকাউন্টের জন্য আলাদা Cloud প্রজেক্ট ও client বানানোও যায়, তবে একই client এ একাধিক test user যোগ করাই সহজ (সর্বোচ্চ ১০০ জন test user)।

### C. শুধু token নবায়ন

```powershell
rclone config reconnect gdrive:
```

---

## ৭. সমস্যা সমাধান

| সমস্যা | কারণ ও সমাধান |
|---|---|
| `rclone` is not recognized | PowerShell পুনরায় খুলুন, বা [PATH রিফ্রেশ](#যদি-rclone-কমান্ড-না-পাওয়া-যায়) করুন |
| `Access blocked: ... access_denied` | অ্যাপ Testing মোডে, Gmail Test users এ নেই। যোগ করুন |
| `invalid_client` | নতুন client চালু হতে সময় লাগে। ৫-১০ মিনিট পর চেষ্টা করুন |
| `invalid_grant` / token expired | ৭ দিনের মেয়াদ শেষ। `rclone config reconnect gdrive:` চালান |
| `Google Drive API has not been used...` | Cloud Console এ Google Drive API **Enable** করুন |
| `rateLimitExceeded` | অনেক দ্রুত অনুরোধ। `--tpslimit 5` যোগ করে চেষ্টা করুন |
| Shared Drive দেখা যায় না | কনফিগে "Configure as Shared Drive" এ `y` দিন, বা `rclone config` → edit |
| ব্রাউজার খোলে না | টার্মিনালের `http://127.0.0.1:53682/auth?state=...` লিংক ম্যানুয়ালি খুলুন |

আরও তথ্য:

- rclone Google Drive ডকুমেন্টেশন: https://rclone.org/drive/
- নিজের client id বানানোর অফিসিয়াল গাইড: https://rclone.org/drive/#making-your-own-client-id

---

## ৮. নিরাপত্তা ও .gitignore

`rclone.conf` এ আপনার Drive এর অ্যাক্সেস token থাকে। যে এটি পাবে সে আপনার Drive এ ঢুকতে পারবে।

এই repo তে `.gitignore` ফাইল রাখুন এবং তাতে লিখুন:

```gitignore
rclone.conf
*.conf
.env
*.secret
```

নিয়মগুলো:

- `client_secret` **repo তে লিখবেন না**। password manager এ রাখুন
- `rclone.conf` কখনো commit করবেন না
- ভুল করে secret প্রকাশ হলে Google Cloud Console → **Clients** → `rclone` এ গিয়ে secret **reset** করুন, তারপর `rclone config` এ নতুনটা বসান
- আর প্রয়োজন না থাকলে অ্যাক্সেস বাতিল করুন: https://myaccount.google.com/permissions

---

## দ্রুত চেকলিস্ট (নতুন সেটআপের জন্য)

- [ ] `winget install Rclone.Rclone`, তারপর নতুন PowerShell
- [ ] Google Cloud প্রজেক্ট তৈরি
- [ ] Google Drive API Enable
- [ ] Auth Platform সেটআপ (External)
- [ ] OAuth client তৈরি (Desktop app)
- [ ] Test users এ Gmail যোগ
- [ ] `rclone config` → `n` → `gdrive` → `drive`
- [ ] Client ID ও secret পেস্ট, scope `1`
- [ ] ব্রাউজারে লগইন ও Allow
- [ ] `rclone lsd gdrive:` দিয়ে টেস্ট
