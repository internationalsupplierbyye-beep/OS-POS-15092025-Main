ဟုတ်ပါတယ်—အောက်မှာ **README (မြန်မာ)** ကို တိုတိတကျ ထုတ်ပေးထားပါတယ်။ ဒီစာကို `README.md` အဖြစ် သိမ်းပြီး သုံးနိုင်ပါတယ်။

---

# POS Cloud – Online Shop (v3, Light Theme)

MMK/THB/USD စနစ်၊ Staff/Delivery စာရင်း၊ အော်ဒါသိမ်းခြင်း၊ Invoice Preview/Print, JPEG ထုတ်ခြင်း၊ **Cloud Settings (Realtime)**၊ **Email+Password Login + Role (admin/staff)**၊ **Reports + CSV Export (Customer data ပါ)** ကိုပါဝင်စေတဲ့ single-file POS စနစ်။

## 1) ပြင်ဆင်ရန်အချက်များ

### 1.1 Source

* `index.html` – အလုပ်လုပ်တဲ့ဖိုင်တစ်ဖိုင်တည်း
* `README.md` – ဒီအကြောင်းအရာ

### 1.2 Firebase Project

* Firebase Console မှာ Project တစ်ခုဖန်တီးပါ
* **Authentication → Sign-in method → Email/Password** ကို enable လုပ်ပါ
* **Authentication → Users** ထဲမှာ အသုံးပြုသူများ (admin/staff) ကို register လုပ်ပါ

### 1.3 Firebase Config

`index.html` ဖိုင်ထဲရှိ

```js
const firebaseConfig = { ... }
```

ကို မင်း project ရဲ့ Config values နဲ့ ပြောင်းထည့်ပါ
(Found at: Project settings → General → Your apps → Config).

---

## 2) Role & Settings Data Structure

### 2.1 Roles (admin / staff)

* Collection: `roles`
* Document ID: **Firebase UID** (user.uid)
* Document data:

```json
{ "role": "admin" }  // သို့မဟုတ် "staff"
```

### 2.2 Global Settings (Delivery & Staff lists)

* Collection: `settings`
* Document ID: `global`
* Data:

```json
{
  "deliveryCompanies": ["Kerry Express","J&T","BeeXpress"],
  "staffIds": ["ST-0001","ST-0002","ST-0003"]
}
```

> Admin သာ UI ထဲက **Settings** မှတစ်ဆင့် realtime ပြင်သိမ်းလို့ရပါတယ်။

---

## 3) Firestore Rules (Admin သာ အကုန်လုပ်/ Staff သာ order create)

**ଆဝင် before**: Rules တွေကို သင့် use-case အတိုင်းပြုပြင်နိုင်ပါတယ်။ အောက်က rule sample က role doc ကိုဖတ်ပြီး Admin/Staff ခွင့်ပြုချက် ခွဲထားပေးပါတယ်။ (Cloud Firestore → Rules)

```rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Everyone signed-in can read their own role doc
    match /roles/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      // Admin promotes via console; no public write
      allow write: if false;
    }

    // Global settings – read for all signed-in, write for admin only
    match /settings/global {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
                   get(/databases/$(database)/documents/roles/$(request.auth.uid)).data.role == "admin";
    }

    // Orders – staff/admin can create; admin can read/list/query; staff cannot list all
    match /orders/{orderId} {
      // Create by any signed-in (staff or admin)
      allow create: if request.auth != null;

      // Read individual order: admin can read all; staff can read their own createdBy only (optional)
      allow read: if request.auth != null && (
        get(/databases/$(database)/documents/roles/$(request.auth.uid)).data.role == "admin"
        || (resource.data.by == request.auth.uid) // if you store 'by' on create
      );

      // Update/Delete: admin only
      allow update, delete: if request.auth != null &&
        get(/databases/$(database)/documents/roles/$(request.auth.uid)).data.role == "admin";
    }
  }
}
```

> **မှတ်ချက်**
> `index.html` ရဲ့ order create code ထဲမှာ `by: auth.currentUser.uid` ထည့်ထားချင်ရင် create payload ထဲပေါင်းထည့်နိုင်ပါတယ် (rules ထဲက own-read logic သုံးချင်သော်).

---

## 4) အလုပ်လုပ်ပုံ

1. `index.html` ကို browser ဖြင့် ဖွင့်ပါ (သို့ GitHub Pages/Firebase Hosting တင်ပါ)
2. အပေါ်ညာ **Cloud: signed-out** ဆိုရင် **Sign in** modal ပြမယ် → Email/Password ဖြည့်ဝင်ပါ
3. Admin account ဝင်ပြီး `Settings` မှာ **Delivery Companies / Staff IDs** စာရင်း ထည့် **Save** လုပ်ပါ
   (Realtime နဲ့ Order form dropdown တွေ update ပြသည်)
4. Order တစ်ခုကို **Staff ID**, Customer, Products, Fees/Discount ထည့်ပြီး **Save to Cloud** လုပ်ပါ
5. **Print / Save PDF** သို့မဟုတ် **Save JPEG** ကိုအသုံးပြုနိုင်သည်
6. Admin မှာ **Reports** → Filter လုပ်ပြီး **Export CSV** (Customer data ပါ) ထုတ်နိုင်သည်

---

## 5) Reports & CSV Export

* Filters: Date range, Staff, Delivery Company, Order Number (contains)
* KPIs: Orders, Revenue (Grand total), Discount
* Tables: By Staff, By Delivery Company, Top Products
* **Export CSV**: Columns

  * `docId,date,staffId,orderNumber,deliveryCompany,subtotal,fee,discount,grand`
  * `customerName,customerPhone,customerFacebook,customerAddress,itemsCount`

CSV ကို Excel ထဲ ဖွင့်နိုင်ပြီး Customer data ကွက်လပ်များပါ ပါဝင်လာပါမယ်။

---

## 6) Hosting Options

* **GitHub Pages**: repo ထဲ `index.html` + `README.md` ထည့်ပြီး Pages enable
* **Firebase Hosting**:

  ```bash
  npm i -g firebase-tools
  firebase login
  firebase init hosting      # public directory: (folder with index.html)
  firebase deploy
  ```

---

## 7) Troubleshooting

* **Save failed: Missing or insufficient permissions**

  * Auth လုပ်ထား/မထား စစ်ပါ
  * Firestore Rules ကို အထက်ပါအတိုင်း update
  * Admin role မှန်/မမှန် (`roles/{uid}`) စစ်ပါ

* **Reports query index required**

  * Console notification မှာ “Create index” link ပေါ်လာမယ် → click OK

* **Settings မထွက်/Dropdown မပြောင်း**

  * `settings/global` document ရှိ/မရှိ
  * DeliveryCompanies / staffIds array format မှန်/မမှန်

* **White page/controls only**

  * `index.html` တစ်ဖိုင်လုံးကို (header + form + invoice + modals + scripts) အပြည့် ထည့်ထားသလား စစ်ပါ

---

## 8) Data Model (Order)

```json
{
  "invoiceNo": "INV-2025-12345",
  "currency": "MMK",
  "date": "2025-09-14T12:34:56.000Z",
  "by": "uid_of_creator (optional)",
  "staffId": "ST-0001",
  "orderNumber": "SHOP-00001",
  "orderType": "Delivery",
  "deliveryCompany": "Kerry Express",
  "tracking": "TRK123456789TH",
  "deliveryFee": 3500,
  "discountAmt": 1200,
  "sellerInfo": "Your Shop, 09xxxx",
  "customer": {
    "name": "Mg Aung Gyi",
    "phone": "09....",
    "facebook": "fb link",
    "address": "Address..."
  },
  "note": "Happy birthday…",
  "items": [
    {"code":"P001","name":"Converse","color":"Red","size":"42","qty":1,"price":35000}
  ],
  "totals": {"subtotal":35000,"fee":3500,"disc":1200,"grand":37300}
}
```

---

## 9) UI Shortcuts

* **New Order** – state reset (invoice no. refresh)
* **Currency** – MMK/THB/USD ဖော်မတ်ပြောင်းပေးသည်
* **Save JPEG** – invoice preview ကို အဖြူနောက်ခံနှင့် JPG ထုတ်ပေးသည်
* **Settings/Reports** – Admin only (role = admin)

---

