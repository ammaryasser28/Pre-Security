> **الهدف من الـ Section ده:**  
> تفهم الفرق الحاسم بين الـ Login Shell والـ Non-Login Shell وقت استخدام `su`، وتعرف تستخدمه بأمان سواء عشان تبقى root أو تجرب صلاحيات مستخدم تاني أو تنفذ أمر واحد بس بصلاحيات مختلفة.


## Table of Contents

- [Pre-Flight Checklist](#pre-flight-checklist)
- [Key Uses of su](#key-uses-of-su)
- [Syntax](#syntax)
- [Login vs Non-Login Shell](#login-vs-non-login-shell)
- [Common Options](#common-options)
- [Step-by-Step Usage](#step-by-step-usage)
- [Tips](#tips)
- [Post-Lab Checklist](#post-lab-checklist)
- [Security Notes (SOC / Blue Team)](#security-notes-soc--blue-team)

---

## Pre-Flight Checklist

- [ ] عارف الفرق بين `su -` و `su` قبل ما تنفذ أي أمر
- [ ] عندك كلمة سر المستخدم الهدف (أو root) جاهزة
- [ ] الجهاز اللي هتجرب عليه بيئة اختبار (VM/Lab)
- [ ] متأكد من اسم المستخدم اللي هتتحول له بالظبط

---

## Key Uses of su

- **Become root**: Gain administrative privileges to perform system-wide tasks
- **Test permissions**: Switch to a regular user account (e.g., service accounts like `www-data`)
- **Run commands as another user**: Execute a single command with the privileges of another user

---

## Syntax

```bash
su [OPTION] [USER]
```

- `OPTION` → command-line options to control behavior
- `USER` → the username you want to switch to
- Must be run as **root** or with **sudo**

---

## Login vs Non-Login Shell

أهم مفهوم في الـ Runbook ده هو الفرق بين **Login Shell** و **Non-Login Shell**.

### `su - [username]` (Login Shell – Recommended)

- الشرطة `-` هي اختصار لـ `--login`
- Loads the full environment of the target user (`.bashrc`, `.profile`)
- Sets the PATH and switches to the user's home directory

**Examples**:

```bash
# Become root
su -

# Become a regular user
su - user
```

### `su [username]` (Non-Login Shell)

- Switches user ID only
- Keeps your current environment and PATH
- Does not change the current directory

> [!WARNING]
> **Risk**: استخدام Non-Login Shell كـ root ممكن يخليك تنشئ ملفات في المكان الغلط أو تنفذ أوامر باستخدام PATH غلط - لأن الـ Environment بتاعك القديم (بتاع المستخدم الأصلي) هو اللي لسه شغال، مش بتاع root.

> [!IMPORTANT]
> **Recommendation**: استخدم `su -` دايمًا إلا لو عندك سبب محدد فعلاً لاستخدام Non-Login Shell.

---

## Common Options

| Command | Behavior |
|---|---|
| `su -` | Switch to root with full login shell (**Recommended**) |
| `su` | Switch to root but keep your current environment (**Not recommended**) |
| `su - [user]` | Switch to `[user]` with login shell (**Recommended**) |
| `su [user]` | Switch to `[user]` but keep your environment |
| `su -c "command"` | Execute a single command as root (with root's environment) and exit |
| `su -p [user]` | Preserve your current environment while switching to `[user]` |
| `-l` or `--login` | Full flag for login shell (`-` equivalent) |
| `-s /bin/sh` | Specify a different login shell |

---

## Step-by-Step Usage

### Step 1: Switch to Root

```bash
sudo su
```

- Enter the root password when prompted
- Gain superuser privileges until you exit the session

> [!CHECKPOINT]
> تحقق: نفذ `whoami` بعد التحول - المفروض يرجع `root`. لو رجع اسم المستخدم القديم، حاجة غلط.

### Step 2: Switch to Another User

```bash
su username
```

- Enter the target user's password
- Example:

```bash
su shivansh260
```

بيحولك لمستخدم `shivansh260` والـ Home Directory بتاعه.

### Step 3: Execute a Command as Another User

```bash
sudo su -c "command" -s /bin/bash username
```

- Runs `command` as `[username]` without switching fully
- `-s /bin/bash` specifies the shell to use

**Example**:

```bash
sudo su -c "echo hello" -s /bin/bash shivansh
```

> [!TIP]
> الأمر ده مفيد جدًا في الـ Scripts الأوتوماتيكية لما تحتاج تنفذ أمر واحد بس بصلاحيات مستخدم معين من غير ما تفتح Session كاملة وتقفلها يدويًا.

### Step 4: Preserve Environment Variables

```bash
sudo su -
```

- Use `-` or `--login` to load the target user's full environment
- Ensures PATH and environment variables are correctly set

### Step 5: Simulate a Full Login Shell

```bash
su -l username
```

- Fully loads the target user's environment as if logging in normally
- Recommended for testing user-specific settings and scripts

> [!CHECKPOINT]
> تحقق: نفذ `echo $PATH` و `pwd` بعد أي تحول - لو استخدمت `-` أو `-l`، المفروض تكون في Home Directory بتاع المستخدم الجديد والـ PATH بتاعه هو الشغال. لو مستخدمتش `-`، هتلاقي إنت لسه في نفس المكان والـ PATH القديم.

---

## Tips

1. Always prefer `su -` for safe environment switching
2. Use `su -c "command"` to run single commands as another user
3. Non-login shells should only be used if you understand the risks
4. Useful for system admins, service accounts, and troubleshooting permissions

---

## Post-Lab Checklist

- [ ] تأكدت من هوية المستخدم الحالي بعد كل تحول باستخدام `whoami`
- [ ] استخدمت `su -` مش `su` عادي إلا لو كان في سبب واضح
- [ ] لو نفذت أمر بصلاحيات مستخدم تاني، تأكدت من النتيجة قبل ما تكمل
- [ ] خرجت من أي Session مفتوحة بـ root باستخدام `exit` بعد ما خلصت شغلك
- [ ] لو كنت بتجرب صلاحيات، رجعت لحسابك الأصلي في الآخر

---

## Security Notes (SOC / Blue Team)

> [!IMPORTANT]
> أمر `su` من أكتر الأوامر اللي لازم تتراقب بعناية في أي بيئة، لأنه بيمثل نقطة التحول المباشرة لصلاحيات أعلى (Privilege Escalation Point) - سواء استخدام شرعي من Admin أو استغلال من مهاجم بعد ما يسرق كلمة سر.

| Risk | Description | MITRE ATT&CK Reference |
|---|---|---|
| Root Password Brute Force via su | محاولات متكررة لتخمين كلمة سر root عن طريق `su -` | T1110 - Brute Force |
| Privilege Escalation after Compromise | مهاجم بعد ما يخترق حساب مستخدم عادي، يحاول يستخدم `su` للوصول لـ root لو عرف الباسورد (أو استغل ثغرة) | T1548 - Abuse Elevation Control Mechanism |
| Lateral Movement via su | استخدام `su` للتنقل بين حسابات خدمة (Service Accounts) مختلفة على نفس الجهاز لتوسيع نطاق الوصول | T1078 - Valid Accounts |
| Environment Manipulation (Non-Login Shell Abuse) | استغلال Non-Login Shell عمدًا للاستفادة من متغيرات بيئة (Environment Variables) قديمة لتنفيذ كود ضار بصلاحيات أعلى | T1574 - Hijack Execution Flow |

> [!WARNING]
> كل محاولة `su` **ناجحة أو فاشلة** بتتسجل عادةً في `/var/log/auth.log` (على أنظمة Debian/Ubuntu) أو `/var/log/secure` (على أنظمة RHEL/CentOS). عدد كبير من محاولات `su` الفاشلة المتتالية على حساب root هو مؤشر كلاسيكي على **Brute Force Attack**.

### Detection & Best Practices

- مراقبة `/var/log/auth.log` أو `/var/log/secure` لأي محاولات `su` فاشلة متكررة
- تقييد استخدام `su` باستخدام أداة زي **`sudo` مع صلاحيات محددة (Least Privilege)** بدل إعطاء كلمة سر root لعدد كبير من المستخدمين
- تفعيل **`pam_wheel`** لتقييد استخدام `su` للانتقال لـ root على مجموعة محددة بس من المستخدمين الموثوقين
- مراجعة سجلات التحول بين المستخدمين بانتظام للتأكد من إن كل استخدام لـ `su` له سبب موثق ومعروف

> [!TIP]
> لو لاحظت في الـ Logs مستخدم عادي عمل `su` بنجاح لـ root في وقت غير معتاد (زي نص الليل) أو من جلسة SSH غريبة، ده يستاهل تحقيق فوري - خصوصًا لو الحساب ده مالوش تاريخ سابق في استخدام صلاحيات إدارية.

---

## Summary

- `su` بيسمحلك تتحول لمستخدم تاني (أو root) مؤقتًا، أو تنفذ أمر واحد بصلاحياته
- **الفرق الجوهري**: `su -` (Login Shell) بيحمل الـ Environment الكامل بتاع المستخدم الجديد وبينقلك لـ Home Directory بتاعه، بينما `su` العادي (Non-Login Shell) بيسيبك في نفس الـ Environment والمكان القديم
- **التوصية الأساسية**: استخدم `su -` دايمًا إلا لو عندك سبب واضح لعكس ده
- من ناحية الـ SOC: `su` نقطة مراقبة أساسية للـ **Privilege Escalation (T1548)** و **Brute Force (T1110)**، ومراجعة `/var/log/auth.log` أو `/var/log/secure` ضرورية لاكتشاف أي محاولات مشبوهة
