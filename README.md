# Study Buzz — backend + website starter

This project includes the existing Study Buzz HTML page and a Node.js backend that implements the API routes used by that page. It uses only built-in Node.js modules; no `npm install` is required.

## Features implemented
- Student registration with SMS OTP verification (requires Twilio Verify configuration)
- Email/password login for existing accounts
- Phone-based SMS OTP login for verified mobile numbers
- Password hashing with Node.js `scrypt`
- Signed, expiring bearer tokens
- Persistent JSON database for users, courses, enrolments and doubts
- Free sample courses and course enrolment
- Student doubt submission and teacher/admin replies
- Admin overview, user role/status management
- Basic auth rate limiting and request validation
- `/api/health` endpoint for a quick server check

## Important limitations
- This is a starter application, not a finished production LMS.
- SMS OTP UI and backend endpoints are implemented using Twilio Verify. **Real SMS will not send until the owner configures valid Twilio Verify credentials** (`TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_VERIFY_SERVICE_SID`) in the server/hosting environment. The provider account may require billing/phone destination permissions. OTP flows expire after about five minutes; pending flows are cleared if the server restarts.
- Paid-course checkout is not implemented. Paid enrolment deliberately returns a message until a payment gateway is connected.
- Video streaming/storage, password reset, email verification, push notifications, backups and multi-instance database locking are not implemented.
- The JSON database is suitable for a small demo/test deployment, not high-traffic production use. Use a managed database before a public launch.
- Public deployment must use HTTPS and persistent storage. Keep secrets out of source control.

## Run on a computer
Requires Node.js 18 or newer.

```sh
node --check server.js
node server.js
```

Open `http://localhost:3000`. The server creates `data/study-buzz-db.json` automatically and seeds three free sample courses.

## Configure SMS OTP (Twilio Verify)

1. Create a Twilio account and a Verify Service in the Twilio console.
2. Set these server environment variables; never put them in HTML or commit them to source control:
   - `TWILIO_ACCOUNT_SID`
   - `TWILIO_AUTH_TOKEN`
   - `TWILIO_VERIFY_SERVICE_SID`
3. Restart the server. `GET /api/health` should report `"smsOtpConfigured":true`.
4. Test registration with a mobile number you control, then enter the OTP actually received by SMS. Indian numbers may need to be entered in international `+91` format; 10-digit Indian numbers are normalized automatically.

Without these secrets, the app intentionally returns a clear configuration error and does not fake a sent OTP. SMS charges, account verification, geographic permissions and provider policies are controlled by Twilio.

## Create the first admin
Set both environment variables before the first start. Password must be at least 12 characters. Example in macOS/Linux:

```sh
ADMIN_EMAIL=admin@example.com ADMIN_PASSWORD='Use-A-Long-Unique-Password!' JWT_SECRET='replace-with-a-long-random-secret-at-least-32-characters' NODE_ENV=production node server.js
```

On Windows PowerShell:

```powershell
$env:ADMIN_EMAIL='admin@example.com'
$env:ADMIN_PASSWORD='Use-A-Long-Unique-Password!'
$env:JWT_SECRET='replace-with-a-long-random-secret-at-least-32-characters'
$env:NODE_ENV='production'
node server.js
```

The admin account is created only if that email does not already exist. Do not use the example password literally.

## Test
With the server running, open `http://localhost:3000/api/health`. It should return JSON with `"ok":true`. Then open the home page, configure Twilio Verify to test SMS OTP registration/login, enrol in a free course, and submit a doubt. Password login remains available for existing accounts created through the original password-registration API. Admin endpoints require the configured admin account.

## Mobile-only note
A local HTML file opened from Android File Manager cannot run this server by itself. You need a Node.js-capable hosting service or a computer running Node.js. On hosting, set the environment variables in the host dashboard and enable persistent storage for the `data` directory. Then open the hosted website URL in Chrome. Do not expose the admin password or JWT secret.
