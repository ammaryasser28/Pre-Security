> **الهدف من الـ Section ده:**  
> تقدر تحذف مستخدمين من Linux بأمان وبالطريقة الصح، تفرق بين الحذف العادي والحذف الكامل (`-r`)، وتعرف امتى تستخدم `-f` بحذر شديد.


## Table of Contents

- [Pre-Flight Checklist](#pre-flight-checklist)
- [What is userdel?](#what-is-userdel)
- [Syntax](#syntax)
- [Common Options](#common-options)
- [Step-by-Step Usage](#step-by-step-usage)
- [Notes for System Administrators](#notes-for-system-administrators)
- [Quick Reference](#quick-reference)
- [Post-Lab Checklist](#post-lab-checklist)
- [Security Notes (SOC / Blue Team)](#security-notes-soc--blue-team)

---

## Pre-Flight Checklist

- [ ] عندك صلاحيات `root` أو `sudo` على الجهاز
- [ ] الجهاز اللي هتجرب عليه بيئة اختبار (VM/Lab)، مش سيرفر Production
- [ ] متأكد 100% من اسم المستخدم اللي هتحذفه (مفيش رجوع بعد الحذف!)
- [ ] عملت **Backup** لأي ملفات مهمة جوه الـ Home Directory بتاع المستخدم لو محتاجها
- [ ] تأكدت إن المستخدم مش عليه Processes أو Services حساسة شغالة حاليًا

> [!WARNING]
> عملية الحذف بـ `userdel` **غير قابلة للتراجع (Irreversible)** خصوصًا مع `-r`. اتأكد من اسم المستخدم مرتين قبل التنفيذ.

---

## What is userdel?

- Deletes a user account from the system
- Removes references to the username in system files like `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/gshadow`
- Can remove home directories and mail spools associated with the user
- Offers various options for force deletion, SELinux cleanup, or working in chroot environments

---

## Syntax

```bash
userdel [options] username
```

- `options`: Modify the behavior of deletion
- `username`: Name of the user to delete
- Must be run as **root** or with **sudo**

---

## Common Options

| Option | Purpose |
|---|---|
| (none) | حذف حساب المستخدم فقط، من غير المساس بالـ Home Directory أو Mail Spool |
| `-f` | Forcefully delete a user, حتى لو كان عامل Login حاليًا |
| `-r` | Delete user along with their home directory and mail spool |
| `-h` | Display quick help message |
| `-R /path/to/chroot` | Apply changes within a specific chroot directory |
| `-Z` | Remove SELinux user mapping |
| `--help` | Show general syntax and all options |

---

## Step-by-Step Usage

### Step 1: Delete a User

```bash
sudo userdel username
```

- Deletes the user account from the system
- Replace `username` with the actual account name
- User cannot be deleted without administrative privileges (`sudo` required)

> [!CHECKPOINT]
> تحقق: نفذ `getent passwd username` بعد الحذف - المفروض ميرجعش أي نتيجة، وده يأكد إن الحساب اتشال من `/etc/passwd`.

### Step 2: Forcefully Delete a User

```bash
sudo userdel -f neuser
```

- Forces deletion even if the user is logged in
- Removes home directory and mail spool regardless of ownership

> [!WARNING]
> استخدام `-f` بيقدر يسبب **حالة نظام غير مستقرة (Inconsistent State)** لو المستخدم كان عنده Processes شغالة وقت الحذف. استخدمه بحذر شديد وبس لما يكون فعلاً ضروري.

### Step 3: Delete User with Home Directory and Mail Spool

```bash
sudo userdel -r newuser2
```

- Deletes user account and all files in their home directory
- Mail spool is also deleted
- Files on other file systems must be removed manually

> [!IMPORTANT]
> `-r` بتحذف بس الملفات اللي جوه الـ **Home Directory** والـ **Mail Spool**. أي ملفات للمستخدم موجودة على أماكن تانية في النظام (زي `/opt` أو مجلدات مشاريع خارج الـ Home) لازم تتحذف يدويًا بنفسك.

### Step 4: Display Help Message

```bash
userdel -h
```

بيوريك مرجع سريع للـ Syntax والخيارات المتاحة.

### Step 5: Apply Changes in a CHROOT_DIR

```bash
sudo userdel -R /path/to/chroot username
```

- Uses configuration files from the specified chroot directory
- Useful in containers, recovery environments, or isolated systems

### Step 6: Remove SELinux User Mapping

```bash
sudo userdel -Z newuser2
```

- Deletes SELinux mappings associated with the user account
- Ensures complete removal from SELinux-enabled systems

> [!NOTE]
> لو النظام بيستخدم **SELinux**، لازم تستخدم `-Z` عشان تضمن إن الـ Mapping بتاع المستخدم اتشال بالكامل. لو نسيتها، ممكن يفضل أثر (Residual Mapping) للمستخدم في إعدادات SELinux حتى بعد حذف الحساب نفسه.

### Step 7: General Help Option

```bash
userdel --help
```

بيوريك الـ Syntax العام وكل الخيارات المتاحة - مفيد لو نسيت أي Flag أو طريقة استخدام.

---

## Notes for System Administrators

- Deleting user accounts is critical for security and system cleanup
- Use `-r` for full cleanup to remove home directories and mail spools
- Use `-f` cautiously to remove users who may be logged in
- SELinux users should also use `-Z` to remove security mappings

---

## Quick Reference

| Command | Purpose |
|---|---|
| `sudo userdel username` | Delete user account only |
| `sudo userdel -f username` | Force delete (even if logged in) |
| `sudo userdel -r username` | Delete user + home directory + mail spool |
| `sudo userdel -R /path username` | Apply deletion within a chroot directory |
| `sudo userdel -Z username` | Remove SELinux user mapping |
| `userdel -h` / `userdel --help` | Show help |

---

## Post-Lab Checklist

- [ ] المستخدم اختفى من `getent passwd username`
- [ ] الـ Home Directory اتشال لو استخدمت `-r` (تأكد بـ `ls /home/`)
- [ ] لو استخدمت `-Z`، تأكد إن SELinux Mapping اتشال (لو النظام بيدعم SELinux)
- [ ] راجعت باقي النظام للتأكد من عدم وجود ملفات متبقية للمستخدم على File Systems تانية غير الـ Home
- [ ] لو المستخدم كان جزء من مجموعات (Groups) مشتركة، تأكد إن باقي الأعضاء متأثروش

---

## Security Notes (SOC / Blue Team)

> [!IMPORTANT]
> `userdel` مش بس أداة إدارية عادية - في سياق الأمن، حذف المستخدمين (أو **عدم** حذفهم في الوقت الصح) بيمثل جزء أساسي من دورة حياة إدارة الحسابات (Account Lifecycle Management)، وأي تقصير فيه بيسيب أبواب مفتوحة.

| Risk | Description | MITRE ATT&CK Reference |
|---|---|---|
| Orphaned/Stale Accounts | عدم حذف حسابات الموظفين اللي سابوا الشركة أو المقاولين المؤقتين بيسيب حسابات صالحة (Valid Accounts) ممكن تُستغل | T1078 - Valid Accounts |
| Anti-Forensics via userdel | مهاجم بعد ما يستخدم حساب مؤقت في الهجوم، يقدر يحذفه بـ `userdel -r` عشان يمسح آثاره ويصعب التتبع | T1070 - Indicator Removal |
| Incomplete Cleanup (Missing -Z) | نسيان `-Z` في بيئات SELinux بيسيب Mappings متبقية ممكن تستغل أو تسبب Confusion في السياسات الأمنية | T1548 - Abuse Elevation Control Mechanism |
| Force Deletion of Active Sessions | استخدام `-f` بشكل غير مبرر ممكن يقطع Session شرعي فجأة، وده ممكن يستخدم كنوع من الـ Denial of Service الداخلي لو تم بسوء نية | T1531 - Account Access Removal |

> [!WARNING]
> **Account Access Removal (T1531)** هي تقنية معروفة بيستخدمها المهاجمين مش بس لتغطية آثارهم، لكن كمان كتكتيك تخريبي - زي حذف حسابات المستخدمين الشرعيين عشان يمنعهم من الوصول أثناء هجوم Ransomware مثلاً. أي استخدام غير متوقع لـ `userdel` على حسابات نشطة لازم يترفع فورًا كـ Alert حرج.

### Detection & Best Practices

- مراقبة أي تنفيذ لـ `userdel` على السيرفرات عن طريق **Audit Logs** (`auditd`, `/var/log/auth.log`)، خصوصًا استخدام `-f` أو `-r`
- ربط عملية حذف المستخدمين بعملية رسمية موثقة (Offboarding Process) مرتبطة بالموارد البشرية، مش أوامر يدوية عشوائية
- مراجعة `/etc/passwd` و `/etc/group` بشكل دوري لاكتشاف أي حسابات قديمة (Stale Accounts) كان المفروض تتحذف من زمان
- التأكد من تطبيق `-Z` في كل عمليات الحذف على أنظمة بتستخدم SELinux كجزء من الإجراء القياسي

> [!TIP]
> لو لاحظت في الـ Logs إن حساب معين اتعمله `useradd` وبعدها بوقت قصير جدًا (دقائق أو ساعات) اتعمله `userdel -r`، ده نمط مثير للشك جدًا - ممكن يكون مهاجم استخدم حساب مؤقت للتنفيذ وبعدين حذفه لمحو آثاره. راجع الـ Logs المرتبطة بنشاط الحساب ده في الفترة القصيرة دي فورًا.
