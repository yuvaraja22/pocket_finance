# Pocket Finance

A privacy-focused personal finance tracker with client-side encryption.

## 🔐 Security & Data Protection

Pocket Finance takes your financial data privacy seriously. Here's how your data is protected:

---

### Standard Mode (UID-Based Security)

When you sign in with Google **without enabling PIN encryption**, your data is protected through:

| Layer | Protection |
|-------|------------|
| **Authentication** | Google OAuth 2.0 - Only you can access your account |
| **Database Isolation** | Each user has their own Firestore collection (`/users/{uid}/transactions`) |
| **Transport Security** | All data transmitted over HTTPS/TLS encryption |
| **Encryption Key** | Your Firebase UID is used to encrypt data via AES-256 |

#### How It Works:
1. You sign in with Google → Firebase generates a unique UID
2. Your UID becomes the encryption key
3. All transaction data is encrypted before being stored in Firestore
4. Only requests authenticated with your Google account can read your data

#### Security Level: 🟡 **Good**
- Your data is encrypted and isolated
- Firebase/Google admins could theoretically decrypt (they have access to your UID)
- If someone gains access to your Google account, they can access your data

---

### End-to-End Encryption Mode (PIN-Based)

When you enable **PIN encryption**, you get true end-to-end encryption (E2EE):

| Layer | Protection |
|-------|------------|
| **Zero-Knowledge** | Only YOU know the decryption key (your PIN) |
| **Client-Side Encryption** | Data is encrypted in your browser BEFORE leaving your device |
| **AES-256** | Military-grade encryption standard |
| **No Server Access** | Firebase/Google cannot read your data - they only see encrypted blobs |

#### How It Works:
1. You set a 6-digit PIN (stored only on your device)
2. Your PIN becomes the encryption key (not stored anywhere on the server)
3. Transaction data is encrypted with AES-256 using your PIN **before** upload
4. Server receives and stores only encrypted data (`{ isEncrypted: true, payload: "encrypted_blob" }`)
5. When you access your data, it's decrypted locally using your PIN

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│   Your Device   │         │    Internet     │         │    Firebase     │
│                 │         │                 │         │                 │
│  Plain Data     │         │                 │         │                 │
│       ↓         │         │                 │         │                 │
│  PIN + AES-256  │         │                 │         │                 │
│       ↓         │  HTTPS  │                 │  Store  │                 │
│  Encrypted ────────────────────────────────────────▶  │  Encrypted      │
│  Blob           │         │                 │         │  Blob Only      │
└─────────────────┘         └─────────────────┘         └─────────────────┘
```

#### Security Level: 🟢 **Excellent**
- Even Firebase admins cannot read your data
- Resistant to server-side breaches
- Only vulnerable if someone knows your PIN AND has your device

---

### ⚠️ Important Considerations

#### PIN Mode Warning
> **If you forget your PIN, your data is UNRECOVERABLE.**  
> There is no "forgot PIN" option because we never store your PIN.

#### Local Storage
- When not signed in, data is stored in browser's localStorage (unencrypted)
- Signing in syncs and encrypts your data to the cloud

#### Best Practices
1. **Use PIN mode** for sensitive financial data
2. **Remember your PIN** - write it down in a secure location
3. **Don't share your PIN** with anyone
4. **Use a unique PIN** - don't reuse PINs from other services

---

## 🛠 Technical Details

### Encryption Library
- **CryptoJS** (AES-256) - [https://cryptojs.gitbook.io](https://cryptojs.gitbook.io)

### Data Flow

```javascript
// Encryption (before save)
const encrypted = CryptoJS.AES.encrypt(JSON.stringify(data), userKey).toString();

// Decryption (after fetch)  
const decrypted = JSON.parse(
  CryptoJS.AES.decrypt(encrypted, userKey).toString(CryptoJS.enc.Utf8)
);
```

### Storage Structure

```
Firestore Database
└── users/
    └── {userId}/
        ├── transactions/
        │   ├── {transactionId}: { isEncrypted: true, payload: "...", date: "..." }
        │   └── ...
        └── settings/
            ├── security: { usePin: true/false }
            └── categories: { ... }
```

---

## 📱 Features

- ✅ Track income and expenses
- ✅ Categorize transactions
- ✅ Monthly analytics with charts
- ✅ Export PDF reports
- ✅ Offline support (localStorage)
- ✅ Cloud sync with Google account
- ✅ Client-side encryption (Standard & E2EE)
- ✅ PWA - Install as mobile app
- ✅ Responsive design (Mobile, Tablet, Desktop)

---

## 🚀 Getting Started

1. Open the app in your browser
2. (Optional) Sign in with Google for cloud sync
3. (Optional) Enable PIN encryption for E2EE
4. Start tracking your finances!

---

*Built with ❤️ for privacy-conscious users*
