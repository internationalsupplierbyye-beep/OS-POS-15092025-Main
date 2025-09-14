POS Cloud – v3 (Cloud Settings + Roles)
====================================

1) Authentication
   - Enable Email/Password.
   - Create users (admin/staff).

2) Firestore Collections
   - roles/{UID} -> { role: "admin" | "staff" }
   - settings/global -> { deliveryCompanies: [..], staffIds: [..] }

3) Rules (Publish)
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /orders/{docId} {
      allow read, write: if isAdmin();
      allow create: if isStaff();
    }

    match /roles/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if isAdmin();
    }

    match /settings/{docId} {
      allow read: if request.auth != null;
      allow write: if isAdmin();
    }
  }
}
function isAdmin() {
  return request.auth != null &&
    get(/databases/(default)/documents/roles/$(request.auth.uid)).data.role == "admin";
}
function isStaff() {
  return request.auth != null &&
    get(/databases/(default)/documents/roles/$(request.auth.uid)).data.role == "staff";
}

4) Run
   - Open with local server or deploy (and add domain to Authorized domains).
   - Sign in -> If admin, you can Save. Everyone can Reload (read) from cloud.

Firebase Console → Firestore →

Collection = settings

Document = global

Fields =

deliveryCompanies (Array) → "Kerry Express", "J&T" …

staffIds (Array) → "ST-0001", "ST-0002" …

👉 မထည့်ရသေးလို့ အလွတ်ပဲ ပြနေတယ်။
