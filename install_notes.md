# Install notes

## Installing from the test apt repository

In order to use the preview build for R34, packages must be obtained from
<https://pkgs-test.nginx.com> instead of <https://pkgs.nginx.com>.

### Configure APT

Replace the apt source in /etc/apt/sources.list.d/nginx-plus.list:

```shell
sed -i 's/pkgs\./pkgs-test\./g' /etc/apt/sources.list.d/nginx-plus.list
```

Update the fqdn of the apt repository in /etc/apt/apt.conf.d/90pkgs-nginx:

```shell
sed -i 's/pkgs\./pkgs-test\./g' /etc/apt/apt.conf.d/90pkgs-nginx
```

Replace the existing nginx repo SSL cert and key with the ones associated with
your NGINX license (obtain these from MyF5 Downloads):

i.e.

```shell
mkdir /etc/nginx/ssl/bak
mv /etc/nginx/ssl/*.{crt,key} /etc/nginx/ssl/bak

cp ~/nginx-repo-A-S00017390.crt /etc/nginx/ssl/nginx-repo.crt
cp ~/nginx-repo-A-S00017390.key /etc/nginx/ssl/nginx-repo.key
```

Run apt-update and apt-show nginx-plus to confirm availability of the R34
test package:

```shell
apt update

apt show nginx-plus | grep Version:
```

## Licensing and Usage Reporting issue

NGINX Plus R34 introduces a new vrsion string format. This format is not yet
supported by the licensing/usage report API endpoint, so we must first install
and license NGINX Plus R33. After installing and successfully licensing R33, we
must then uninstall R33 and install R34. R34 will use the leftover state file
from R33, satisfying the licensing endpoint's schema requirements. This will not
be necessary at GA.

Nginx state file location:

```shell
/var/lib/nginx/state/nginx-mgmt-state
```

### Installation workaround

```shell
apt -y install nginx-plus=33-2~jammy
```

```shell
cp ~/license.jwt /etc/nginx
```

```shell
# enable reporting in /etc/nginx/nginx.conf
sed -i \
    -e 's/^#mgmt/mgmt/' \
    -e 's/\ #usage_report/\ usage_report/' \
    -e 's/\ #license_token/\ license_token/' \
    -e 's/\ #enforce_initial_report/\ enforce_initial_report/' \
    -e '$ s/^#//' \
    /etc/nginx/nginx.conf

# enable debug logging in /etc/nginx/nginx.conf
sed -i \
    's/error\.log\ notice/error\.log\ debug/' \
    /etc/nginx/nginx.conf
```

```shell
systemctl start nginx-plus
```

```shell
tail -f /var/log/nginx/error.log | grep 'usage\ report\ was\ sent'
```

```shell
systemctl stop nginx
```

```shell
apt -y remove nginx-plus && apt install nginx-plus # remove r33 and install r34
```

```shell
systemctl start nginx
```

```shell
tail -f /var/log/nginx/error.log | grep 'usage\ report\ was\ sent'
```
