<!-- DevSponsors Badges -->
<p align="center">
  <a href="https://devsponsors.github.io"><img src="https://img.shields.io/badge/DevSponsors-Verified_OSS-6366f1?style=for-the-badge&logo=github" alt="DevSponsors Verified"></a>
  <a href="https://devsponsors.github.io"><img src="https://img.shields.io/badge/Sponsor-DevSponsors_Hub-emerald?style=for-the-badge&logo=github-sponsors" alt="DevSponsors Sponsor"></a>
  <a href="https://devsponsors.github.io/mediakit.html"><img src="https://img.shields.io/badge/Infrastructure-DevSponsors_Cloud-ec4899?style=for-the-badge&logo=server" alt="DevSponsors Cloud"></a>
</p>

# RealityGhost v3.6.1
### Dual-Mode Xray Reality (XHTTP / TCP) with Stealth HTTPS Subscription

RealityGhost یک اسکریپت **production‑grade، ایمن و stealth‑محور** برای راه‌اندازی و مدیریت Xray Reality (VLESS) است که بر اساس **مشاهدات میدانی، لاگ‌های عملیاتی و گزارش‌های واقعی از نحوه فیلترینگ در ایران** طراحی شده است.

این پروژه مخصوص استفاده طولانی‌مدت در شرایط فشار شدید DPI و Active Probing نوشته شده، نه صرفاً تست.

---

## ✨ ویژگی‌های کلیدی

- Dual Transport واقعی: `XHTTP (packet-up)` و `TCP`
- سوییچ زنده Transport بدون reinstall
- Reality استاندارد (pbk فقط client-side)
- Subscription کاملاً HTTPS و استتارشده
- SAFE Rotation غیرمخرب
- طراحی‌شده برای محیط ایران

---

## 🔁 چرا Rotation هر ۳ روز انجام می‌شود؟ (بخش بسیار مهم)

### 🔍 مشاهده میدانی چه چیزی را نشان داد؟

بر اساس:
- گزارش‌های کاربران متعدد
- لاگ‌های NGINX + Xray
- تست‌های عملی در بازه‌های زمانی مختلف
- رفتار DPI و Active Probing در ایران

الگوی زیر به‌صورت **واضح** مشاهده شده است:

> **Reality endpointهایی که بیش از ~۷۲ ساعت بدون تغییر باقی می‌مانند، به‌صورت تدریجی شناسایی و degrade می‌شوند.**

این شناسایی معمولاً نه به‌صورت قطع فوری، بلکه به شکل‌های زیر رخ می‌دهد:
- افزایش latency غیرعادی
- packet loss تصادفی
- timeout در TLS handshake
- قطع کامل فقط برای برخی ISPها
- اختلال فقط روی fingerprint یا shortId خاص

---

### 🎯 دقیقاً چه چیزهایی شناسایی می‌شوند؟

برخلاف تصور عمومی:
- **IP یا Domain اولین هدف نیست**
- **Reality Key هم معمولاً بلافاصله block نمی‌شود**

آنچه به‌صورت عملی target می‌شود:
- **TLS Fingerprint ثابت**
- **shortId یکتا و بدون تغییر**
- **الگوی handshake تکراری در زمان**

یعنی اگر:
- fp ثابت بماند
- shortId همیشه یکی باشد
- و handshake pattern تغییر نکند

سیستم فیلترینگ به مرور endpoint را به‌عنوان **Reality tunnel فعال** علامت‌گذاری می‌کند.

---

### ✅ راه‌حل RealityGhost چیست؟

RealityGhost دقیقاً بر همین اساس طراحی شده:

#### SAFE Rotation (پیش‌فرض)
هر ۳ روز به‌صورت خودکار:
- TLS Fingerprint تغییر می‌کند
- shortId جدید **اضافه** می‌شود (نه جایگزین)
- shortIdهای قدیمی حفظ می‌شوند (تا N عدد)

نتیجه:
- الگوی handshake تغییر می‌کند
- endpoint از دید DPI «جدید» به نظر می‌رسد
- **هیچ کلاینت فعالی قطع نمی‌شود**

---

### ❌ چرا Reality Key هر بار rotate نمی‌شود؟

Reality Key rotate:
- باعث drop فوری تمام کلاینت‌ها می‌شود
- از نظر امنیتی **ضرورتی ندارد**
- از نظر فیلترینگ **اولویت target نیست**

به همین دلیل:
- key rotation فقط به‌صورت **opt‑in و آگاهانه** انجام می‌شود.

---

### ⏱ چرا دقیقاً هر ۳ روز؟

- کمتر از ۲۴ ساعت: unnecessary churn
- بیشتر از ۳–۴ روز: ریسک شناسایی بالا می‌رود
- ۷۲ ساعت sweet‑spot عملی بین:
  - stealth
  - stability
  - client uptime

به همین دلیل cron پیش‌فرض به‌شکل زیر است:

```
0 5 */3 * * root realityghost manual-rotate
```

---

## 🔐 SAFE vs HARD Rotation

| نوع | چه چیزی تغییر می‌کند | Drop کلاینت |
|----|--------------------|-------------|
| SAFE (default) | fingerprint + append shortId | ❌ |
| HARD (ROTATE_KEYS=1) | Reality Key | ✅ |
| HARD (ROTATE_PATH=1) | XHTTP Path | ✅ |

---

## 📦 معماری کلی

Client → TLS (google.com SNI)
→ NGINX :443 (ssl_preread)
→ Xray Reality (127.0.0.1:8444)

Subscription:
→ HTTPS واقعی (127.0.0.1:8443)

---

## ⚙️ نصب

```bash
bash realityghost.sh install
```

---

## 🧭 مدیریت

```bash
realityghost manage
```

شامل:
- مشاهده لینک‌ها
- QR Code
- Manual Rotate
- Switch Transport
- Update UUID
- Logs / Stats
- Uninstall

---

## 🧠 فلسفه طراحی

RealityGhost بر اساس این اصل نوشته شده:

> **در ایران، پایداری مهم‌تر از تغییرات تهاجمی است.**

به همین دلیل:
- Rotate غیرمخرب
- تغییر تدریجی
- بدون pattern ثابت
- بدون رفتار مشکوک

---

## 📜 License
MIT
