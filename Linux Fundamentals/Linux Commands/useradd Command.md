
> **الهدف من الـ Section ده:**  
> تقدر تنشئ وتدير مستخدمين على Linux باستخدام `useradd` بثقة، تفهم الفرق بينه وبين `adduser`، وتعرف تتحقق (Verify) من كل خطوة قبل ما تكمل للي بعدها.




## Table of Contents

- [Pre-Flight Checklist](#pre-flight-checklist)
- [useradd vs adduser](#useradd-vs-adduser)
- [What Happens When You Run useradd?](#what-happens-when-you-run-useradd)
- [Syntax](#syntax)
- [Step-by-Step Usage](#step-by-step-usage)
- [Verify User](#verify-user)
- [Quick Reference](#quick-reference)
- [Post-Lab Checklist](#post-lab-checklist)
- [Security Notes (SOC / Blue Team)](#security-notes-soc--blue-team)

---

## Pre-Flight Checklist

- [ ] عندك صلاحيات `root` أو `sudo` على الجهاز
- [ ] الجهاز اللي هتجرب عليه بيئة اختبار (VM/Lab)، مش سيرفر Production
- [ ] عارف اسم المستخدم (Username) اللي هتنشئه مقدمًا
- [ ] فاتح Terminal جاهز للتنفيذ

---

## useradd vs adduser

| Aspect | `useradd` | `adduser` |
|---|---|---|
| Type | Native binary | Perl script |
| Interaction Level | Low-level, less interactive | User-friendly, interactive |
| Behavior | Requires explicit flags for most settings | Asks for password, full name, etc. automatically |
| Relationship | - | Uses `useradd` internally |
| Recommended Use | Servers and scripts | Manual/interactive desktop use |

> [!IMPORTANT]
> للشغل الاحترافي وفي الامتحانات، الأفضلية دايمًا لـ `useradd` لأنه الأمر الأساسي (Native)، وفهمه بيديك تحكم كامل في كل تفصيلة بتحصل وقت إنشاء المستخدم.

---

## What Happens When You Run useradd?

الأمر `useradd` بيعدل ملفات النظام دي مباشرة:

| File | Purpose |
|---|---|
| `/etc/passwd` | stores basic user information |
| `/etc/shadow` | stores encrypted passwords |
| `/etc/group` | stores primary group info |
| `/etc/gshadow` | stores group permissions |
| `/home/username` | home directory (if created) |

---

## Syntax

```bash
useradd [options] username
```

**Requirements**: لازم تنفذه كـ `root` أو باستخدام `sudo`.

---

## Step-by-Step Usage

### Step 1: Create a Basic User

```bash
sudo useradd test_user
```

- Creates a new user
- No password or home directory by default
- User cannot log in until a password is set

**Set password**:

```bash
sudo passwd test_user
```

> [!CHECKPOINT]
> تحقق: نفذ `id test_user` - المفروض يرجعلك UID وGID للمستخدم الجديد، وده يأكد إن الإنشاء نجح قبل ما تكمل.

### Step 2: Specify Home Directory

```bash
sudo useradd -d /home/test_user test_user
```

- Sets a custom home directory
- Does not create it automatically unless `-m` is used

**Recommended**:

```bash
sudo useradd -m -d /home/test_user test_user
```

> [!WARNING]
> لو نسيت `-m`، الأمر هيحدد المسار بس **من غير ما ينشئ المجلد فعليًا**. لو المستخدم حاول يعمل Login، ممكن يواجه مشاكل لأن الـ Home Directory مش موجود أصلاً.

### Step 3: Specify User ID (UID)

```bash
sudo useradd -u 1234 test_user
```

- Assigns a specific UID
- Useful for syncing across servers, LDAP, or NFS

> [!NOTE]
> الـ UID لازم يكون فريد (Unique)، وعادةً بيكون أكبر من 1000 للمستخدمين العاديين (الأرقام الأقل من كده محجوزة عادةً لحسابات النظام).

### Step 4: Specify Group ID (GID)

```bash
sudo useradd -g 1000 test_user
```

بيضيف المستخدم لمجموعة موجودة بالفعل عن طريق الـ GID بتاعها.

### Step 5: Create User Without Home Directory

```bash
sudo useradd -M test_user
```

- Useful for service accounts or restricted users
- Enhances system security

### Step 6: Set Account Expiry Date

```bash
sudo useradd -e 2020-05-30 test_user
```

- User cannot log in after this date
- Useful for temporary accounts or trainees

### Step 7: Add Comment / Description

```bash
sudo useradd -c "This is a test user" test_user
```

- Adds a description in `/etc/passwd`
- Typically includes full name, job title, or employee ID

### Step 8: Change Login Shell

```bash
sudo useradd -s /bin/sh test_user
```

- Assigns a specific login shell (default is `/bin/bash`)
- Useful for restricted shells or service users

### Step 9: Set Unencrypted Password (⚠️ Not Recommended)

```bash
sudo useradd -p test_password test_user
```

> [!WARNING]
> **Password is plain text**. استخدم `sudo passwd test_user` بدل منها للأمان.

### Step 10: Display Help

```bash
sudo useradd --help
```

أو:

```bash
man useradd
```

---

## Verify User

بعد أي خطوة إنشاء أو تعديل، تأكد دايمًا باستخدام:

```bash
id test_user
```

```bash
getent passwd test_user
```

> [!CHECKPOINT]
> تحقق نهائي: قارن نتيجة `getent passwd test_user` مع الإعدادات اللي قصدت تحطها (Home Directory، Shell، UID/GID). لو أي حاجة مش متطابقة، ارجع للخطوة اللي عملتها وتأكد من الـ Flags اللي استخدمتها.

---

## Quick Reference

| Flag | Purpose |
|---|---|
| `useradd` | Create a user |
| `passwd` | Always set password after creating a user |
| `-m` | Create home directory |
| `-u` | Set UID |
| `-g` | Set primary GID |
| `-s` | Set shell |
| `-e` | Set expiry date |
| `-M` | Do not create home directory |
| `-c` | Add comment/description |
| `-d` | Specify home directory path |

---

## Post-Lab Checklist

- [ ] المستخدم اتعمل بنجاح وظهر في `getent passwd`
- [ ] كلمة السر اتظبطت باستخدام `passwd` (مش عن طريق `-p`)
- [ ] الـ Home Directory اتعمل فعليًا لو كان مطلوب (`-m` استخدمت)
- [ ] الـ Shell والـ UID/GID مطابقين للمطلوب
- [ ] لو المستخدم مؤقت، تاريخ الـ Expiry اتظبط صح
- [ ] لو كان حساب اختبار بس، امسحه بعد الانتهاء باستخدام `userdel -r test_user`

---

## Security Notes (SOC / Blue Team)

> [!IMPORTANT]
> أمر `useradd` بيعدل ملفات حساسة جدًا زي `/etc/passwd` و `/etc/shadow`، وأي استخدام غير متوقع ليه على سيرفر Production لازم يترفع كـ Alert فوري.

| Risk | Description | MITRE ATT&CK Reference |
|---|---|---|
| Unauthorized Account Creation | مهاجم بعد ما يخترق جهاز، ينشئ مستخدم جديد له UID عادي عشان يستخدمه كـ Backdoor دائم | T1136.001 - Create Account: Local Account |
| Plaintext Password via `-p` | استخدام `-p` بيسجل الباسورد بشكل ممكن يتلاحظ في الـ Shell History أو الـ Logs | T1552.003 - Unsecured Credentials: Bash History |
| UID 0 Assignment (Privilege Escalation) | تعيين UID = 0 لمستخدم جديد بيدي له صلاحيات root كاملة من غير ما يكون اسمه "root" | T1078.003 - Valid Accounts: Local Accounts |
| Hidden Service Accounts | إنشاء حسابات بـ `-M` (من غير Home Directory) وShell غريب لتقليل الأثر الظاهر وتسهيل الإخفاء | T1136 - Create Account |

> [!TIP]
> لو بتحقق في جهاز Linux، شغل الأمر ده عشان تدور على أي مستخدم بـ UID = 0 غير `root` نفسه (مؤشر خطير جدًا على Privilege Escalation):
> ```bash
> awk -F: '$3 == 0 {print $1}' /etc/passwd
> ```
> لو رجعلك أي اسم غير `root`، ده يستاهل تحقيق فوري.

### Detection & Best Practices

- مراقبة أي تنفيذ لأمر `useradd` على سيرفرات Production عن طريق Audit Logs (`auditd` أو `/var/log/auth.log`)
- مراجعة `/etc/passwd` دوريًا للتأكد من عدم وجود مستخدمين غير موثقين أو UID = 0 مكرر
- تجنب استخدام `-p` نهائيًا في أي بيئة حقيقية، والاعتماد فقط على `passwd` التفاعلي
- تفعيل تنبيهات (Alerts) فورية على أي تعديل لملفات `/etc/passwd` أو `/etc/shadow` خارج نافذة الصيانة المعتادة
