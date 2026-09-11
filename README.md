# Folderum2ElectricBoogaloo

I wanted to prove you could make a forum without a database.

Then I removed the files.

I don't know why.

Users are folders. Posts are folders. Votes are folders. Sessions are folders. Passwords are plaintext folder names. Even the PHP source code is encoded into folder names.

We removed SQL injection by removing SQL.

Then we removed files.

## Getting it

GitHub doesn't preserve empty folders.

Unfortunately, **the empty folders are the program**.

So the repository contains:

```text
README.md
Folderum.zip
```

If you use GitHub's **Download ZIP**, you'll get something named roughly:

```text
Folderum2ElectricBoogaloo.zip
```

Extract that first.

Inside is the repository, including:

```text
Folderum.zip
```

**`Folderum.zip` is the actual Folderum directory structure.**

Yes, the software is zipped because GitHub refuses to acknowledge my database schema.

## Requirements

Debian/Ubuntu:

```bash
sudo apt update
sudo apt install -y php-cli unzip
```

Optional, if you want to look at what I've done:

```bash
sudo apt install -y tree
```

PHP 8+.

That's basically it.

## Install

After downloading and extracting `Folderum2ElectricBoogaloo.zip`, find the included:

```text
Folderum.zip
```

Then:

```bash
cd /var/www/html
sudo unzip /path/to/Folderum.zip
```

You want the resulting structure at:

```text
/var/www/html/folderum/
```

If it extracted as `Folderum` instead:

```bash
sudo mv /var/www/html/Folderum /var/www/html/folderum
```

Give PHP permission to create more database rows, also known as directories:

```bash
sudo chown -R www-data:www-data /var/www/html/folderum
sudo chmod -R 750 /var/www/html/folderum
```

Now verify the entire point of this project:

```bash
find /var/www/html/folderum -type f
```

It should print nothing.

Or:

```bash
find /var/www/html/folderum -type f | wc -l
```

should return:

```text
0
```

Good.

The software has no files.

## Run

```bash
cd /var/www/html/folderum

sudo -u www-data env \
FOLDERUM_ROOT="/var/www/html/folderum" \
FOLDERUM_LISTEN="0.0.0.0:8080" \
php -r '
$p=getenv("FOLDERUM_ROOT")."/program";
$a=array_values(array_filter(
    scandir($p),
    fn($x)=>$x!=="."&&$x!==".."&&is_dir($p."/".$x)
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

Open:

```text
http://YOUR-SERVER-IP:8080/
```

Port `8080` is separate from your normal Apache site on port `80`.

`Ctrl+C` stops it.

The first registered user becomes admin.

## How

The PHP source is stored like this:

```text
program/
├── 000000_...
├── 000001_...
├── 000002_...
└── ...
```

The launch command reads those folder names, reconstructs the PHP source and executes it.

Then the reconstructed PHP creates more folders.

For example:

```text
data/
└── users/
    └── greg/
        ├── password_beer123/
        └── role_admin/
```

Yes, that's the password.

Don't use a real one.

## Administration

Ban Greg:

```bash
sudo -u www-data mkdir /var/www/html/folderum/data/users/greg/banned
```

Unban Greg:

```bash
sudo -u www-data rmdir /var/www/html/folderum/data/users/greg/banned
```

Make Greg admin:

```bash
sudo -u www-data rmdir /var/www/html/folderum/data/users/greg/role_user
sudo -u www-data mkdir /var/www/html/folderum/data/users/greg/role_admin
```

Lock a thread:

```bash
sudo -u www-data mkdir /var/www/html/folderum/data/forum/CATEGORY/threads/THREAD_ID/locked
```

Unlock it:

```bash
sudo -u www-data rmdir /var/www/html/folderum/data/forum/CATEGORY/threads/THREAD_ID/locked
```

We removed SQL injection by replacing the admin console with `mkdir`.

## Debugging

Check port 8080:

```bash
sudo ss -ltnp | grep ':8080'
```

Test write permissions:

```bash
sudo -u www-data mkdir /var/www/html/folderum/data/test
sudo -u www-data rmdir /var/www/html/folderum/data/test
```

Look at the database:

```bash
tree /var/www/html/folderum/data
```

Look at the source code:

```bash
tree /var/www/html/folderum/program
```

`tree` is the database client.

`find` is the query language.

`mkdir` is apparently the ORM.

## Security

No.

This is a joke.

Passwords are deliberately plaintext directory names. Use throwaway credentials and don't expose this to anything important.

I wanted to prove that a forum could exist without SQL.

We removed SQL injection by removing SQL.

Then I wondered if it could exist without files.

Unfortunately, yes.

I proved my point.

I'm tired.
