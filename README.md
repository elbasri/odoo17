# تثبيت وإعداد أودو 17
 
## المتطلبات الأساسية:
- خادم لينكس يعمل بنظام **أوبونتو 22.04** بحد أدنى 2 جيجابايت من الذاكرة.
- مستخدم لديه صلاحيات الجذر أو مستخدم بدون صلاحيات جذر مع صلاحيات `sudo`.


## الخطوة 1: تحديث النظام
قم بتحديث جميع الحزم إلى أحدث الإصدارات باستخدام الأمر التالي:
```bash
sudo apt update && sudo apt upgrade -y
```

## الخطوة 2: تثبيت بايثون والأدوات اللازمة
ستحتاج إلى تثبيت بايثون 3 و pip وبعض الأدوات اللازمة لتشغيل Odoo. قم بتنفيذ الأمر التالي:
```bash
sudo apt install python3 python3-pip python3-dev build-essential libpq-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev libldap2-dev libsasl2-dev libtiff5-dev libjpeg8-dev libopenjp2-7-dev liblcms2-dev libblas-dev libatlas-base-dev
```

## الخطوة 3: تثبيت أدوات NPM و CSS
لتثبيت Node.js و npm لإدارة حزم JavaScript، استخدم الأوامر التالية:
```bash
sudo apt install npm
sudo ln -s /usr/bin/nodejs /usr/bin/node
sudo npm install -g less less-plugin-clean-css
```

## الخطوة 4: تثبيت Wkhtmltopdf
Wkhtmltopdf هي أداة لتحويل صفحات الويب إلى ملفات PDF. قم بتثبيتها باستخدام الأوامر التالية:
```bash
sudo apt install wget
sudo wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.bionic_amd64.deb
sudo apt install ./wkhtmltox_0.12.6-1.bionic_amd64.deb
```

## الخطوة 5: تثبيت PostgreSQL
PostgreSQL هو خادم قاعدة البيانات الذي سيستخدمه Odoo. قم بتثبيته وتفعيله باستخدام الأوامر التالية:
```bash
sudo apt install postgresql
sudo systemctl start postgresql
sudo systemctl enable postgresql
```
للتحقق من حالة PostgreSQL، استخدم الأمر:
```bash
sudo systemctl status postgresql
```

## الخطوة 6: إنشاء مستخدمين لأودو و PostgreSQL
ستحتاج إلى إنشاء مستخدم لأودو و PostgreSQL. قم بتنفيذ الأوامر التالية:
```bash
sudo adduser --system --home=/opt/odoo --group odoo
sudo su - postgres
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo
exit
```

## الخطوة 7: تثبيت وتكوين Odoo ERP 17
قم باستنساخ مستودع Odoo 17 وتثبيت الحزم المطلوبة باستخدام الأوامر التالية:
```bash
sudo su - odoo
git clone https://www.github.com/odoo/odoo --depth 1 --branch 17.0 --single-branch .
python3 -m venv odoo-venv
source odoo-venv/bin/activate
pip install -r requirements.txt
deactivate
```
ثم قم بإنشاء دليل السجلات والمكونات الإضافية:
```bash
mkdir /var/log/odoo
mkdir /opt/odoo/custom-addons
```

## الخطوة 8: إنشاء ملف إعدادات Odoo 17
لإنشاء وتحرير ملف إعدادات Odoo، استخدم الأمر:
```bash
sudo nano /etc/odoo.conf
```
أضف الأسطر التالية إلى ملف الإعدادات:
```ini
[options]
admin_passwd = YourStrongPasswordHere
db_host = False
db_port = False
db_user = odoo
db_password = False
addons_path = /opt/odoo/addons,/opt/odoo/custom-addons
logfile = /var/log/odoo/odoo.log
```
ثم احفظ وأغلق الملف.

## الخطوة 9: إنشاء ملف وحدة نظام systemd لأودو
لإدارة خدمة Odoo، قم بإنشاء ملف وحدة systemd باستخدام الأمر:
```bash
sudo nano /etc/systemd/system/odoo.service
```
أضف المحتوى التالي إلى الملف:
```ini
[Unit]
Description=Odoo
Documentation=https://www.odoo.com
[Service]
# Ubuntu/Debian convention:
Type=simple
User=odoo
ExecStart=/opt/odoo/odoo-venv/bin/python3 /opt/odoo/odoo-bin -c /etc/odoo.conf
[Install]
WantedBy=multi-user.target
```
بعد ذلك، قم بإعادة تحميل daemon وبدء خدمة Odoo باستخدام الأوامر التالية:
```bash
sudo systemctl daemon-reload
sudo systemctl start odoo
sudo systemctl enable odoo
```
للتحقق من حالة الخدمة، استخدم الأمر:
```bash
sudo systemctl status odoo
```

## الخطوة 10: الوصول إلى Odoo 17
للوصول إلى Odoo، افتح متصفح الويب الخاص بك وانتقل إلى:
```bash
http://YourServerIPAddress:8069
```