# 📁 Folderum: ZERO REGULAR FILES EDITION

> A PHP forum for people who looked at a filesystem and thought:
>
> **"what if the source code was also directories?"**

Welcome to **Folderum: Zero Regular Files Edition**.

This is the version where I finally stopped pretending.

The installed forum tree contains:

- no `.php` files
- no `.txt` files
- no `.json`
- no SQLite database
- no CSS file
- no JavaScript file
- no config file
- no session files
- no post files
- no user files
- no files

**There are only directories.**

The application source itself is base64url-encoded, split into chunks, and stored in directory names under:

```text
folderum/program/
```

The forum data is also stored entirely as directories under:

```text
folderum/data/
```

The only regular file in this ZIP is **this README**, which intentionally lives *outside* the `folderum/` runtime directory.

So after installation:

```bash
find /var/www/html/folderum -type f
```

should output:

```text
absolutely fucking nothing
```

This is no longer filesystem-backed software.

This is an **executable inode sculpture**.

---

# ⚠️ SERIOUSLY: THIS IS A JOKE

This project is intentionally ridiculous.

Do not use it for anything important.

Do not expose it to the public Internet with real users.

Do not reuse a real password.

Passwords are deliberately stored in **plaintext directory names**.

If somebody registers:

```text
username: greg
password: beer123
```

Folderum may create:

```text
data/
└── users/
    └── greg/
        ├── password_beer123/
        └── role_user/
```

Authentication is approximately:

> "does the directory named after your password exist?"

This is terrible security.

That is intentional.

Use throwaway credentials.

---

# 🍺 Requirements

This edition does **not** need Apache to run.

It uses PHP itself as a tiny HTTP server built into the directory-encoded application.

You need:

- Linux
- PHP 8+
- PHP CLI
- `unzip`
- optionally `tree`
- enough free inodes to commit crimes against computing

On Debian / Ubuntu:

```bash
sudo apt update
sudo apt install -y php-cli unzip
```

Optional:

```bash
sudo apt install -y tree
```

Check PHP:

```bash
php -v
```

You want PHP 8 or newer.

---

# 📦 Installation

This README assumes you want Folderum at:

```text
/var/www/html/folderum
```

Extract the ZIP somewhere temporary:

```bash
cd /tmp
unzip /path/to/folderum-zero-files-with-readme.zip
```

You should get:

```text
folderum-zero-files-package/
├── README.md
└── folderum/
    ├── data/
    └── program/
```

Move the runtime tree into `/var/www/html`:

```bash
sudo rm -rf /var/www/html/folderum
sudo mv /tmp/folderum-zero-files-package/folderum /var/www/html/folderum
```

You may also keep this README somewhere sane:

```bash
sudo cp /tmp/folderum-zero-files-package/README.md /var/www/html/FOLDERUM-README.md
```

That README copy is deliberately **outside** the runtime folder so this remains true:

```bash
find /var/www/html/folderum -type f
```

Expected output:

```text
nothing
```

If that command prints a filename, somewhere along the way somebody accidentally introduced civilization.

---

# 🔐 Permissions

Folderum stores users, sessions, categories, threads, posts, votes and settings by creating directories.

So the account running PHP needs permission to create directories under:

```text
/var/www/html/folderum/data
```

If you are using Debian / Ubuntu and want to run the server as `www-data`:

```bash
sudo chown -R www-data:www-data /var/www/html/folderum
sudo chmod -R 750 /var/www/html/folderum
```

Test it:

```bash
sudo -u www-data mkdir /var/www/html/folderum/data/permission_test
sudo -u www-data rmdir /var/www/html/folderum/data/permission_test
```

No error?

Congratulations.

Your database server has passed the industry-standard `mkdir` connectivity test.

---

# 🧪 Verify The Zero-File Claim

Run:

```bash
find /var/www/html/folderum -type f
```

There should be no output.

Count regular files:

```bash
find /var/www/html/folderum -type f | wc -l
```

Expected:

```text
0
```

Count directories instead:

```bash
find /var/www/html/folderum -type d | wc -l
```

That number is your application, database, configuration, source code and emotional damage.

---

# 🚀 Starting Folderum

Folderum's PHP source code is stored as directory names under:

```text
/var/www/html/folderum/program/
```

To run it, PHP must:

1. scan those directories
2. sort them
3. remove the numeric prefixes
4. concatenate the base64url chunks
5. decode them
6. `eval()` the reconstructed application
7. start listening for HTTP requests

Yes.

That is the boot process.

Run this:

```bash
cd /var/www/html/folderum

sudo -u www-data env \
FOLDERUM_ROOT="/var/www/html/folderum" \
FOLDERUM_LISTEN="0.0.0.0:8080" \
php -r '
$p=getenv("FOLDERUM_ROOT")."/program";

$a=array_values(array_filter(
    scandir($p),
    fn($x)=>
        $x!=="." &&
        $x!==".." &&
        is_dir($p."/".$x)
));

sort($a,SORT_NATURAL);

$s="";

foreach($a as $n){
    $s.=explode("_",$n,2)[1];
}

$s.=str_repeat("=",(4-strlen($s)%4)%4);

eval(base64_decode(strtr($s,"-_","+/")));
'
```

You should see something like:

```text
Folderum zero-file edition on http://0.0.0.0:8080
Installed tree contains directories only. Ctrl+C to stop.
```

Then open:

```text
http://YOUR-SERVER-IP:8080/
```

Registration:

```text
http://YOUR-SERVER-IP:8080/register
```

Login:

```text
http://YOUR-SERVER-IP:8080/login
```

Press:

```text
Ctrl+C
```

to stop it.

---

# 🌐 Will Port 8080 Break My Normal Website On Port 80?

No.

Ports are separate.

You can have:

```text
Apache / normal website   → port 80
Folderum                  → port 8080
```

at the same time.

For example:

```text
http://YOUR-SERVER/
```

can still be your normal site, while:

```text
http://YOUR-SERVER:8080/
```

is Folderum.

Check whether 8080 is already in use:

```bash
sudo ss -ltnp | grep ':8080'
```

No output means it is probably free.

Check both ports:

```bash
sudo ss -ltnp | grep -E ':80|:8080'
```

You might see:

```text
LISTEN ... :80    ... apache2
LISTEN ... :8080  ... php
```

The two applications can coexist peacefully despite Folderum's best efforts.

---

# 🔒 Safer Local-Only Launch

The normal launch command above uses:

```text
0.0.0.0:8080
```

which means Folderum listens on every network interface.

If you only want to access it from the server itself, use:

```bash
FOLDERUM_LISTEN="127.0.0.1:8080"
```

instead.

So the beginning becomes:

```bash
sudo -u www-data env \
FOLDERUM_ROOT="/var/www/html/folderum" \
FOLDERUM_LISTEN="127.0.0.1:8080" \
php -r '
```

This is a much better idea for a demo machine.

Remember: plaintext password directories.

---

# 👑 First User Becomes Admin

The first account registered gets:

```text
role_admin/
```

Every later normal account gets:

```text
role_user/
```

Example:

```text
data/
└── users/
    ├── drunkadmin/
    │   ├── password_beer123/
    │   └── role_admin/
    │
    └── greg/
        ├── password_hunter2/
        └── role_user/
```

This is our access-control database.

The database is a hallway containing appropriately named doors.

---

# 🔑 Password Rules

Passwords are stored directly in directory names.

Therefore Folderum only accepts characters which can safely participate in its terrible idea:

```text
A-Z
a-z
0-9
_
-
```

Minimum password length:

```text
6
```

Examples that work:

```text
beer123
hunter2
very_secure_password
my-password-123
```

Examples that do not:

```text
hello world
what/the/fuck
../../etc/passwd
password!
```

We did not eliminate path traversal through sophisticated database design.

We eliminated the database.

---

# 🗂️ Categories

Categories are directories under:

```text
data/forum/
```

Creating a category from the web interface effectively results in:

```text
data/
└── forum/
    └── Technology/
        └── threads/
```

Manually create one:

```bash
sudo -u www-data mkdir -p \
    /var/www/html/folderum/data/forum/Technology/threads
```

That is effectively:

```sql
INSERT INTO categories ...
```

except SQL has been replaced with a syscall and regret.

---

# 🧵 Threads

A thread is another directory:

```text
data/
└── forum/
    └── Technology/
        └── threads/
            └── 0123456789abcdef/
```

Inside it you will find more directories representing things such as:

```text
author_.../
title/
posts/
```

The title itself may be split across multiple encoded directory names.

Why?

Because directory-name length limits attempted to stop me.

They failed.

---

# 📝 Posts

Posts live under a thread:

```text
threads/
└── THREAD_ID/
    └── posts/
        └── POST_ID/
```

A post may resemble:

```text
POST_ID/
├── author_YWxpY2U/
├── body/
│   ├── 000000_SGVsbG8gZnJvbSB0aGUg/
│   ├── 000001_ZmlsZXN5c3RlbQ/
│   └── ...
└── votes/
    ├── up/
    └── down/
```

Folderum reconstructs the body by reading ordered directory names, joining them, and decoding the result.

I implemented `TEXT` using directory entries.

Database engineers hate this one weird trick.

---

# 👍 Votes

Votes are directories.

Alice upvotes a post:

```text
votes/
└── up/
    └── alice/
```

Greg downvotes it:

```text
votes/
└── down/
    └── greg/
```

Score:

```text
number of directories in votes/up
-
number of directories in votes/down
```

No vote table.

No foreign key.

No transaction log.

Just a counting problem.

---

# 🔨 Ban A User Manually

To ban Alice:

```bash
sudo -u www-data mkdir \
    /var/www/html/folderum/data/users/alice/banned
```

Done.

No database query.

No boolean column.

No moderation table.

The existence of:

```text
banned/
```

means Alice is banned.

There is no boolean.

There is only ontology.

The web admin section also exposes ban/unban controls to admins.

---

# 🕊️ Unban A User

Remove the ban directory:

```bash
sudo -u www-data rmdir \
    /var/www/html/folderum/data/users/alice/banned
```

Alice has been forgiven by POSIX.

---

# 👑 Promote A User To Admin Manually

Suppose Alice currently has:

```text
role_user/
```

Remove it:

```bash
sudo -u www-data rmdir \
    /var/www/html/folderum/data/users/alice/role_user
```

Create:

```bash
sudo -u www-data mkdir \
    /var/www/html/folderum/data/users/alice/role_admin
```

Congratulations.

You have implemented RBAC with `mkdir`.

---

# 👤 Demote An Admin

Remove:

```bash
sudo -u www-data rmdir \
    /var/www/html/folderum/data/users/alice/role_admin
```

Create:

```bash
sudo -u www-data mkdir \
    /var/www/html/folderum/data/users/alice/role_user
```

The role-management subsystem is complete.

Somewhere, LDAP felt a disturbance in the Force.

---

# 🔒 Lock A Thread Manually

A lock is represented by a directory called:

```text
locked/
```

Suppose the thread is:

```text
/var/www/html/folderum/data/forum/Technology/threads/0123456789abcdef
```

Lock it:

```bash
sudo -u www-data mkdir \
    /var/www/html/folderum/data/forum/Technology/threads/0123456789abcdef/locked
```

The application sees `locked/` and refuses new replies.

No:

```sql
UPDATE threads SET locked = 1
```

Instead:

```bash
mkdir locked
```

The filesystem has spoken.

---

# 🔓 Unlock A Thread

```bash
sudo -u www-data rmdir \
    /var/www/html/folderum/data/forum/Technology/threads/0123456789abcdef/locked
```

Thread unlocked.

Democracy restored.

---

# 🍺 Sessions Are Folders Too

Login sessions live under:

```text
data/sessions/
```

Conceptually:

```text
sessions/
└── RANDOM_SESSION_ID/
    └── user_YWxpY2U/
```

The browser cookie contains the random session ID.

The server then asks:

> "Does that session directory exist, and which username directory metadata is inside it?"

Even authentication state has been denied the dignity of being a file.

---

# 🔍 Admire The Damage

Install `tree`:

```bash
sudo apt install -y tree
```

Then:

```bash
tree /var/www/html/folderum
```

Users:

```bash
tree /var/www/html/folderum/data/users
```

Forum:

```bash
tree /var/www/html/folderum/data/forum
```

Program source:

```bash
tree /var/www/html/folderum/program
```

The last one is particularly beautiful because it looks like somebody dropped a base64 encoder down a staircase.

---

# 🧬 How The Source Code Works

Normally a PHP project contains:

```text
index.php
auth.php
forum.php
database.php
```

Folderum contains:

```text
program/
├── 000000_CmZ1bmN0aW9u...
├── 000001_ZnVuY3Rpb24...
├── 000002_JGRhdGE...
├── 000003_aWY...
└── ...
```

Each directory name contains:

```text
ORDER_BASE64URLCHUNK
```

The boot command sorts those directories, strips the order prefix, concatenates the chunks, base64-decodes them and sends the reconstructed code into:

```php
eval(...)
```

So:

```bash
cat index.php
```

cannot show you the program.

There is no `index.php`.

To read the source you must effectively ask:

> "filesystem, please describe yourself until PHP happens."

This is the closest I have come to making source code out of architecture.

Literally.

---

# 🧮 Querying The Database

Want all users?

```bash
find /var/www/html/folderum/data/users \
    -mindepth 1 -maxdepth 1 -type d
```

Want banned users?

```bash
find /var/www/html/folderum/data/users \
    -type d -name banned
```

Want admins?

```bash
find /var/www/html/folderum/data/users \
    -type d -name role_admin
```

Want locked threads?

```bash
find /var/www/html/folderum/data/forum \
    -type d -name locked
```

Want the database client?

```bash
find
```

Congratulations.

You have rediscovered SQL as shell commands.

---

# 🩺 Troubleshooting

## Port 8080 is already used

Check:

```bash
sudo ss -ltnp | grep ':8080'
```

Use a different port:

```text
0.0.0.0:8081
```

by changing:

```bash
FOLDERUM_LISTEN="0.0.0.0:8081"
```

Then browse:

```text
http://YOUR-SERVER-IP:8081/
```

---

## Permission denied while registering/posting

Check ownership:

```bash
ls -la /var/www/html/folderum
ls -la /var/www/html/folderum/data
```

Fix:

```bash
sudo chown -R www-data:www-data /var/www/html/folderum
sudo chmod -R 750 /var/www/html/folderum
```

Test:

```bash
sudo -u www-data mkdir \
    /var/www/html/folderum/data/permission_test

sudo -u www-data rmdir \
    /var/www/html/folderum/data/permission_test
```

---

## Nothing happens when I visit port 80

Correct.

This edition listens directly on whatever port you put in:

```text
FOLDERUM_LISTEN
```

The examples use:

```text
8080
```

So visit:

```text
http://YOUR-SERVER-IP:8080/
```

not:

```text
http://YOUR-SERVER-IP/folderum/
```

unless you separately configure Apache as a reverse proxy.

---

## The server stops when I close the terminal

Also correct.

The simple demonstration command runs in the foreground.

That is intentional because this is primarily a joke/demo edition.

For a presentation, keeping the terminal visible is honestly better because people get to watch PHP reconstruct itself from directory names.

---

# 💾 Backup

The database and program are both just directory trees.

Backup:

```bash
sudo tar -czf folderum-inode-disaster.tar.gz \
    /var/www/html/folderum
```

Yes, tar technically creates a file.

The installed application itself still contains none.

Please direct all complaints to `/dev/null`.

---

# 🧹 Reset All Forum Data

Stop Folderum first.

Then:

```bash
sudo rm -rf /var/www/html/folderum/data/users/*
sudo rm -rf /var/www/html/folderum/data/forum/*
sudo rm -rf /var/www/html/folderum/data/sessions/*
```

But note: shell globs do not match hidden entries by default.

This build does not normally create hidden data directories.

After resetting, the next person who registers becomes admin again.

---

# ☢️ Uninstall

When the guilt becomes too much:

```bash
sudo rm -rf /var/www/html/folderum
```

Your inode table can finally begin healing.

---

# 🏆 Folderum Engineering Principles

1. If a boolean can be represented by a directory, make a directory.
2. If a string cannot fit in one directory name, make more directories.
3. If a database could solve the problem, pretend databases were never invented.
4. If `mkdir()` can solve the problem, stop thinking.
5. Missing directory = false.
6. Existing directory = true.
7. Username = directory.
8. Password = somehow also directory.
9. Vote = directory.
10. Ban = directory.
11. Session = directory.
12. Source code = directories.
13. The filesystem is now the ORM.
14. `find` is the query language.
15. `tree` is the admin console.
16. Inodes are basically rows if you disrespect both concepts enough.
17. **EVERYTHING. IS. FOLDERS.**

---

# 🎥 Suggested Demo Sequence

If you are showing this thing to another human:

First prove there are no files:

```bash
find /var/www/html/folderum -type f
```

Then show the program:

```bash
ls /var/www/html/folderum/program | head
```

Then start Folderum with the giant `php -r` bootstrap.

Register:

```text
admin / beer123
```

Then run:

```bash
tree /var/www/html/folderum/data/users
```

Create a category and thread.

Then:

```bash
tree /var/www/html/folderum/data/forum
```

Ban someone using:

```bash
mkdir banned
```

Lock a thread using:

```bash
mkdir locked
```

Finally:

```bash
find /var/www/html/folderum -type f | wc -l
```

and reveal:

```text
0
```

Then stare directly into the camera and say:

> **"We eliminated SQL injection by eliminating SQL. Then we eliminated files."**

---

# Final Warning

Folderum Zero Regular Files Edition is not a serious forum platform.

It is an argument with computer science expressed through `mkdir()`.

Use throwaway credentials.

Do not expose real passwords.

Do not put important information in it.

Do not blame PHP.

PHP did not ask for this.

The filesystem did not ask for this either.

Yet here we are.
