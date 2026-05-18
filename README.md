# 💻 n8n — Code Node দিয়ে Data Cleaning Guide

> Google Sheets থেকে আসা Raw Data কে Code Node দিয়ে Clean করে Gmail এ পাঠানোর গাইড।

🔗 **Code Explain:** [n8n-Code-Node-CodeExplain](https://github.com/Omarmdwasimuddin/n8n-Code-Node-CodeExplain)

---

## 📊 Workflow Overview

```
Google Sheets
(Wrong Structure Data)
        │
        ▼
Google Sheets Trigger
(On row added)
        │
        ▼
Code Node (JavaScript)
[Data Clean করো]
        │
        ▼
Gmail: Send a message
```

---

## 📋 Google Sheets প্রস্তুত করো

### ✅ ধাপ ১ — Google Sheets এ ভুল Structure এ Data দাও

Google Sheets এ এভাবে Data দাও — যেমন নাম বড়-ছোট হাতি মেশানো, Amount এ টাকার চিহ্ন সহ, Email এ Extra Space:

| full_name | email | amount | date | message |
|-----------|-------|--------|------|---------|
| `jOHN doe` | ` John@Gmail.COM ` | `৳1,500.00` | `2026-04-15` | `hello i need a quote` |

![Wrong Structure Google Sheet](https://imgur.com/Hq7HWQm.png)

> 💡 এই এলোমেলো Data কে Code Node দিয়ে পরিষ্কার করা হবে।

---

## ⚙️ Workflow সেটআপ

### ✅ ধাপ ২ — Google Sheets Trigger যোগ করো

- **Nodes Panel** ওপেন করো
- **"Google Sheets"** সার্চ করো এবং ক্লিক করো
- **"On row added"** সিলেক্ট করো
- **Document** যোগ করো

---

### ✅ ধাপ ৩ — Code Node যোগ করো

- **Google Sheets Trigger** এর **`+`** বাটনে ক্লিক করো
- **"code"** সার্চ করো
- **"Code in JavaScript"** সিলেক্ট করো
- নিচের Code টি লিখো

![Code Node](https://imgur.com/pSE5jA3.png)

---

## 🧑‍💻 JavaScript Code

```javascript
const data = $json;

// ১. নাম পরিষ্কার করো
const cleanFull = data.full_name.trim().replace(/\s+/g, ' ');
const parts = cleanFull.split(" ");
const first = parts[0] || "";
const last = parts[1] || "";

// ২. Email পরিষ্কার করো
let email = data.email.trim().toLowerCase();

// ৩. Amount পরিষ্কার করো
const rawAmount = data.amount;
const amountStr = String(rawAmount).trim().replace(/[^0-9.]/g, "");
const amount = parseFloat(amountStr) || 0;

// ৪. Capitalize ফাংশন
function cap(s) {
  return s.charAt(0).toUpperCase() + s.slice(1).toLowerCase();
}

// ৫. নাম ও তারিখ সাজাও
const firstName = cap(first);
const lastName = cap(last);
const rawDate = data.date;
const d = new Date(rawDate);
const niceDate = d.toDateString();

// ৬. Subject ও Message তৈরি করো
const subject = `Quote Request from ${firstName} - Amount ${amount}`;
const msg = data.message || "";
const cleanMsg = msg.charAt(0).toUpperCase() + msg.slice(1);

// ৭. Clean Data Return করো
return { firstName, lastName, email, amount, niceDate, subject, cleanMsg };
```

---

## 📖 Code এর ব্যাখ্যা

### ১. Data নেওয়া
```javascript
const data = $json
```
Google Sheets Trigger থেকে আসা পুরো Row এর Data নেওয়া হচ্ছে।

---

### ২. নাম পরিষ্কার করা

```javascript
const cleanFull = data.full_name.trim().replace(/\s+/g, ' ')
const parts = cleanFull.split(" ")
```

| কাজ | আগে | পরে |
|-----|-----|-----|
| `.trim()` | `" jOHN doe "` | `"jOHN doe"` |
| `.replace(/\s+/g, ' ')` | `"jOHN  doe"` | `"jOHN doe"` |
| `.split(" ")` | `"jOHN doe"` | `["jOHN", "doe"]` |

---

### ৩. Email পরিষ্কার করা

```javascript
let email = data.email.trim().toLowerCase()
```

| আগে | পরে |
|-----|-----|
| `" John@Gmail.COM "` | `"john@gmail.com"` |

---

### ৪. Amount পরিষ্কার করা

```javascript
const amountStr = String(rawAmount).trim().replace(/[^0-9.]/g, "");
const amount = parseFloat(amountStr) || 0;
```

| আগে | পরে |
|-----|-----|
| `"৳1,500.00"` | `1500` |

> `[^0-9.]` — সংখ্যা ও দশমিক বিন্দু ছাড়া সব বাদ দেয়।

---

### ৫. Capitalize ফাংশন

```javascript
function cap(s) {
  return s.charAt(0).toUpperCase() + s.slice(1).toLowerCase();
}
```

| আগে | পরে |
|-----|-----|
| `"jOHN"` | `"John"` |
| `"doe"` | `"Doe"` |

---

### ৬. তারিখ Format করা

```javascript
const d = new Date(rawDate);
const niceDate = d.toDateString();
```

| আগে | পরে |
|-----|-----|
| `"2026-04-15"` | `"Wed Apr 15 2026"` |

---

### ৭. চূড়ান্ত Output

```javascript
return { firstName, lastName, email, amount, niceDate, subject, cleanMsg }
```

পরিষ্কার করা সব Data একসাথে Return হয় — যা Gmail Node এ ব্যবহার করা হবে।

---

## ✉️ Gmail Node সেটআপ

### ✅ ধাপ ৪ — Gmail Node যোগ করো

- **Code in JavaScript** এর **`+`** বাটনে ক্লিক করো
- **"Gmail"** সার্চ করো → **"Send a message"** সিলেক্ট করো
- নিচের মতো Value সেট করো:

| Field | Value |
|-------|-------|
| **To** | `{{ $json.email }}` |
| **Subject** | `{{ $json.subject }}` |
| **Message** | `{{ $json.cleanMsg }}` |

![Gmail Setup](https://imgur.com/wUBidSx.png)

---

### ✅ ধাপ ৫ — Workflow Execute করো

**Execute Workflow** বাটনে ক্লিক করো।

> ✔️ সফল হলে Google Sheets এ নতুন Row যোগ হলে — Data Clean হয়ে Gmail এ পাঠানো হবে।

---

## 📋 Quick Reference

| ধাপ | Node | কাজ |
|-----|------|-----|
| ১ | — | Google Sheets এ Raw Data দাও |
| ২ | Google Sheets Trigger | Row Added হলে চালু হও |
| ৩ | Code (JavaScript) | Data Clean ও Format করো |
| ৪ | Gmail | Clean Data দিয়ে Email পাঠাও |
| ৫ | — | Execute ও যাচাই করো |

### Code Node কী কী করলো:

| Data | আগে (Raw) | পরে (Clean) |
|------|-----------|-------------|
| **নাম** | `jOHN doe` | `John Doe` |
| **Email** | ` John@Gmail.COM ` | `john@gmail.com` |
| **Amount** | `৳1,500.00` | `1500` |
| **তারিখ** | `2026-04-15` | `Wed Apr 15 2026` |
| **Message** | `hello i need...` | `Hello i need...` |

---

*Code Node দিয়ে যেকোনো ধরনের এলোমেলো Data কে সুন্দরভাবে সাজিয়ে Email বা অন্য কাজে ব্যবহার করা যায়।*
