# Study Buzz Android APK project

Yeh Android Studio source project hai. Yeh **ready-made APK nahi** hai. APK build karne se pehle backend website ko HTTPS hosting par deploy karke URL set karna zaroori hai.

## Is project mein kya hai
- Android Studio WebView wrapper with app name Study Buzz
- Internet permission, JavaScript/DOM storage for the web app, back navigation, connection error message
- `backend/` folder mein Study Buzz HTML + Node.js server + smoke test
- SMS OTP endpoints Twilio Verify integration ke saath; real SMS ke liye server secrets zaroori

## APK se pehle zaroori setup
1. `backend/README.md` padhein.
2. Node.js-capable host par `backend/` deploy karein. HTTPS aur persistent storage enable karein.
3. Host environment mein `JWT_SECRET`, `ADMIN_EMAIL`, `ADMIN_PASSWORD` aur Twilio Verify ki `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_VERIFY_SERVICE_SID` set karein. Secrets sirf server ke environment variables mein rakhein, app/HTML mein nahi.
4. Website URL ko browser mein khol kar test karein. `/api/health` mein `ok:true` aur SMS enable karne ke baad `smsOtpConfigured:true` aana chahiye.
5. `app/build.gradle.kts` mein `STUDY_BUZZ_URL` ko apne actual HTTPS website URL se replace karein, example: `"https://studybuzz.example.com"`. Example URL ko as-is na chhodein.

## APK build (Windows laptop)
1. Android Studio install karein: https://developer.android.com/studio
2. Android Studio > Open > is extracted folder `StudyBuzzAndroidProject` ko select karein.
3. Gradle sync complete hone dein. JDK 17 / Android SDK 35 install karne ko kahe to install karein.
4. `app/build.gradle.kts` mein real website URL set karke Gradle Sync karein.
5. Menu: Build > Build Bundle(s) / APK(s) > Build APK(s).
6. Debug APK aam taur par `app/build/outputs/apk/debug/app-debug.apk` mein milegi. Use apne phone mein transfer karke install karein.

## Tests actually run in this workspace
- `node --check backend/server.js` — passed
- `node backend/test/smoke.js` — passed: health endpoint, sample course listing, and safe SMS-not-configured response
- Android APK build/device install — **not run here** (Android SDK/Gradle build environment unavailable)
- Real SMS OTP — **not verified**; requires real Twilio credentials, an eligible destination number and live server deployment.

## Limitations
Payment gateway, live video classes, password reset, push notifications, production database and backups are not implemented. The JSON database is for a small starter/demo, not a high-traffic public launch. Keep the backend secured and use a managed database before broad public use.

## Backend ko Render par online karna (user ke account se)

Yeh project Render deployment blueprint `render.yaml` ke saath diya gaya hai. Main aapke account mein login karke deployment publish nahi kar sakta; hosting account aur SMS provider ki secret keys aapko apne account mein set karni hongi.

1. Is project ko apne GitHub repository mein upload karein. `backend/.env` ya asli secret keys kabhi upload na karein.
2. Render par account banayein/login karein aur **New + > Blueprint** se apni GitHub repository select karein.
3. Render `render.yaml` ko read karke service banane ke settings dikhayega. Review karke deploy karein. Persistent disk hosting plan ke mutabik extra cost le sakti hai; confirm karne se pehle pricing dekhein.
4. Render service ke Environment section mein `ADMIN_EMAIL` aur `ADMIN_PASSWORD` set karein. Admin password unique aur kam-se-kam 12 characters ka rakhein. `JWT_SECRET` auto-generate hone dein.
5. Real SMS OTP ke liye Twilio Verify Service banayein aur `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_VERIFY_SERVICE_SID` ko Render Environment mein set karein. In values ko HTML, GitHub ya chat mein share na karein. Twilio account permissions/billing bhi configure hone chahiye.
6. Deploy complete hone par Render URL ko browser mein kholein, phir `/api/health` par jaayein. Expected JSON mein `ok:true` aana chahiye. SMS setup ke baad `smsOtpConfigured:true` hona chahiye. Yeh sirf configuration check hai; real OTP ko apne phone par send/verify karke hi confirm karein.
7. `app/build.gradle.kts` mein `STUDY_BUZZ_URL` ko Render ka actual HTTPS URL daalein (example: `https://your-service.onrender.com`), phir Android Studio se Gradle Sync aur Build APK karein.

### Zaroori sachchai
- Render par service deploy karne ke liye aapka Render/GitHub account aur approval zaroori hai. Is chat se main aapke accounts create, terms accept, billing approve, ya Twilio secrets set nahi kar sakta.
- JSON file database sirf chhote test ke liye hai. Persistent disk ke bina restart par data reset ho sakta hai; multiple instances ke liye yeh database safe nahi hai. Public production launch se pehle managed database (jaise PostgreSQL), backups, password reset, payment provider, video hosting aur further security review chahiye.
- SMS OTP actual provider credentials ke bina send nahi hota. Fake/test code ko real SMS samajhkar use na karein.


## GitHub se APK build karne ka alternative (Android Studio ke bina)

Yeh workflow GitHub Actions par APK compile kar sakta hai; yeh APK abhi is ZIP ke andar maujood nahi hai.

1. Pehle backend ko Render par deploy karein aur real HTTPS URL lein. Jab tak backend URL nahi milta, workflow APK build nahi karega.
2. GitHub par nayi **private repository** banayein. Is ZIP ko extract karke iske andar ke saare files/folders repository mein upload karein. `.env` ya passwords/keys upload na karein.
3. GitHub repository mein **Settings > Secrets and variables > Actions > Variables > New repository variable** kholein.
4. Variable name `STUDY_BUZZ_URL` aur value Render ka actual HTTPS URL set karein, jaise `https://study-buzz-xxxx.onrender.com` (sirf example; apna real URL use karein).
5. **Actions** tab > **Build Study Buzz APK** > **Run workflow** > **Run workflow**.
6. Workflow green hone par run page ke **Artifacts** section se `Study-Buzz-debug-APK` download karein. ZIP extract karein; `app-debug.apk` Android phone par install ki ja sakti hai. Phone mein zarurat padne par browser/File Manager ko “Install unknown apps” permission deni hogi.

Agar build fail ho, Actions run ka error message dekhein. APK build ho jaana real SMS OTP ka proof nahi hai; OTP tabhi chalega jab Twilio Verify ki asli settings backend hosting par configured hon aur live test successful ho.
