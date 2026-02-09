# Security Configuration Guide

## 🔒 Security Checklist for SpherePay Client Manager

### 1. Deploy Firestore Security Rules (CRITICAL)

The `firestore.rules` file contains rules that:
- ✅ Only allow authenticated users with `@spherepay.co` emails
- ✅ Validate required fields on create/update
- ✅ Deny access to any undefined collections
- ✅ Server-side email domain verification (cannot be bypassed)

**To deploy the rules:**

```bash
# Install Firebase CLI if not already installed
npm install -g firebase-tools

# Login to Firebase
firebase login

# Initialize Firebase in this directory (select existing project)
firebase init

# Deploy only Firestore rules
firebase deploy --only firestore:rules
```

---

### 2. Configure Firebase Console Settings

#### A. Authentication → Settings → Authorized Domains
Go to: https://console.firebase.google.com/project/aggregator-client-manager/authentication/settings

**Remove** any domains you don't need. Keep only:
- `aggregator-client-manager.firebaseapp.com`
- `noflo920.github.io`
- `localhost` (for development only - remove in production)

#### B. Authentication → Sign-in Method
Go to: https://console.firebase.google.com/project/aggregator-client-manager/authentication/providers

**Ensure only Google is enabled.** Disable any other providers:
- ❌ Email/Password
- ❌ Anonymous
- ❌ Phone
- ❌ Other OAuth providers

#### C. Firestore → Rules Tab
Go to: https://console.firebase.google.com/project/aggregator-client-manager/firestore/rules

**Verify the rules match `firestore.rules` file** or deploy them using Firebase CLI.

---

### 3. Restrict API Key in Google Cloud Console (IMPORTANT)

Go to: https://console.cloud.google.com/apis/credentials?project=aggregator-client-manager

**Find your Firebase API key** (starts with `AIzaSy...`) and click to edit it.

#### A. Application Restrictions
Set to **HTTP referrers (websites)** and add:
```
https://noflo920.github.io/*
https://aggregator-client-manager.firebaseapp.com/*
```

For development, optionally add:
```
http://localhost:*/*
http://127.0.0.1:*/*
```

#### B. API Restrictions
Restrict to only these APIs:
- ✅ Identity Toolkit API
- ✅ Token Service API
- ✅ Cloud Firestore API
- ✅ Firebase Installations API

---

### 4. Security Features Already Implemented

#### Client-Side Protections:
- ✅ Google OAuth with `hd` parameter restricting to `spherepay.co`
- ✅ Email domain verification on login and auth state change
- ✅ Auto sign-out for non-spherepay.co users

#### Server-Side Protections (in firestore.rules):
- ✅ Authentication required for all operations
- ✅ Email domain verification using `request.auth.token.email`
- ✅ Field validation for required data
- ✅ Default deny for undefined collections

---

### 5. Testing Security

#### Test 1: Unauthenticated Access
Open browser console on the live site and try:
```javascript
// This should FAIL with permission-denied
db.collection('clients').get().then(console.log).catch(console.error);
```

#### Test 2: Wrong Domain User
Sign in with a non-spherepay.co Google account:
- Should be immediately signed out
- Firestore operations should fail with permission-denied

#### Test 3: Direct Firestore Manipulation
Even with a valid spherepay.co user, try invalid data:
```javascript
// This should FAIL due to field validation
db.collection('clients').add({invalid: 'data'});
```

---

### 6. Additional Recommendations

#### Enable Firebase App Check (Optional but Recommended)
This adds an extra layer of protection against abuse:
1. Go to Firebase Console → App Check
2. Enable reCAPTCHA v3 for web
3. Enforce App Check for Firestore

#### Set Up Monitoring
1. Enable Firebase Analytics
2. Set up alerts for unusual activity in Google Cloud Monitoring
3. Review Firebase Auth logs periodically

#### Regular Security Reviews
- [ ] Review authorized domains quarterly
- [ ] Check for any new sign-in methods enabled
- [ ] Audit Firestore rules when adding new features
- [ ] Monitor for unusual data access patterns

---

## Quick Security Status

| Item | Status |
|------|--------|
| Firestore Rules | ⚠️ Deploy Required |
| API Key Restrictions | ⚠️ Configure in GCP Console |
| Authorized Domains | ⚠️ Review in Firebase Console |
| Sign-in Providers | ⚠️ Verify only Google enabled |
| Client-side Validation | ✅ Implemented |
| Domain Restriction (client) | ✅ Implemented |
| Domain Restriction (server) | ✅ In firestore.rules |

After completing all steps above, your app will be bulletproof! 🛡️
