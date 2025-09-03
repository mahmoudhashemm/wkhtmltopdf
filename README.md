تمام ✅ خليني الخصلك كل الخطوات اللي عملناها من الأول لحد ما التقارير في Odoo 18 اشتغلت بالـ **Google Chrome Headless** بدل wkhtmltopdf. تقدر تاخد الخطوات دي زي README وتحطها على GitHub 👌

---

# 🚀 Odoo 18 PDF Reports with Google Chrome Headless on Docker (Ubuntu 22.04)

## 1. حل مشكلة الشبكة (Outbound traffic + DNS)

* في السيرفر الأساسي كان في قواعد iptables مانعة الحاويات من الاتصال بالإنترنت.
* اتعمل سماح للشبكة الداخلية الخاصة بالحاوية:

```bash
iptables -I DOCKER-USER -s 10.0.1.88/29 -p tcp -m multiport --dports 80,443 -j ACCEPT
iptables -I DOCKER-USER -s 10.0.1.88/29 -p tcp --dport 53 -j ACCEPT
iptables -I DOCKER-USER -s 10.0.1.88/29 -p udp --dport 53 -j ACCEPT
```

* ضبط DNS دائمًا في Docker:
  ملف `/etc/docker/daemon.json`:

```json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

ثم:

```bash
systemctl restart docker
```

---

## 2. تثبيت Google Chrome داخل الحاوية

ادخل الحاوية:

```bash
docker exec -it osloop-18-odoo17-1 bash
```

نزّل Google Chrome Stable:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
dpkg -i google-chrome-stable_current_amd64.deb || apt --fix-broken install -y
```

---

## 3. تثبيت المكتبات الناقصة (dependencies)

```bash
apt-get install -y \
  libnspr4 libnss3 libxss1 libasound2 fonts-liberation \
  libatk-bridge2.0-0 libgtk-3-0 libx11-xcb1 libxcb-dri3-0 \
  libcolord2 libepoxy0 libgdk-pixbuf-2.0-0 \
  libpango-1.0-0 libpangocairo-1.0-0 libpangoft2-1.0-0 \
  libwayland-cursor0 libwayland-egl1 \
  libxcomposite1 libxcursor1 libxdamage1 libxi6 libxinerama1 libxrandr2
```

---

## 4. اختبار Chrome Headless

```bash
google-chrome-stable --version

google-chrome-stable \
  --headless --disable-gpu --no-sandbox \
  --print-to-pdf=/tmp/test.pdf \
  https://www.google.com

ls -lh /tmp/test.pdf
```

لو ظهر الملف → Chrome جاهز ✅

---

## 5. ربط Odoo بالـ Chrome

عدل ملف إعدادات Odoo (`/etc/odoo/odoo.conf` أو system parameters):

```ini
[options]
report.url = http://localhost:8069
report.webkit_binary = /usr/bin/google-chrome-stable --headless --disable-gpu --no-sandbox --disable-software-rasterizer --print-to-pdf
```

ثم أعد تشغيل الحاوية:

```bash
docker restart osloop-18-odoo17-1
```

---

## 6. اختبار من Odoo

* افتح أي تقرير PDF (فاتورة، طلب بيع…).
* في اللوجات هتلاقي زمن التوليد قل بشكل ملحوظ:

  * **wkhtmltopdf** كان \~30 ثانية لـ 80 صفحة.
  * **Chrome headless** بين \~14 ثانية لنفس الحجم (≈ 50% أسرع). 🚀

---

## 📌 ملاحظات

* رسائل الـ `dbus` في اللوج أثناء تشغيل Chrome جوه Docker ممكن تتجاهلها (Warnings مش Errors).
* مشكلة `websocket` على البورت 8072 منفصلة عن التقارير وممكن تتظبط من إعدادات الـ proxy لو محتاج live notifications.

---

تحب أجهزلك نسخة **Markdown جاهزة** (زي README.md) من الخطوات دي تنفع تحطها على GitHub على طول؟
