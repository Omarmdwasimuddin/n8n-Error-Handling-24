# ⚠️ n8n — Error Handling Guide

> n8n এ Error Handling সেটআপ করে Supabase Insert এর সময় Error ধরা ও Success/Failed Email পাঠানোর গাইড।

---

## 📊 Workflow Overview

```
Google Sheets Trigger
(On row added)
        │
        ▼
Supabase: Create a row
[Error হলে → $json.error তৈরি হয়]
        │
        ▼
IF Condition
[$json.error আছে কিনা চেক করো]
        │
        ├── ✅ True (Error আছে)  ──► Gmail: "Failed" Email
        │
        └── ❌ False (Error নেই) ──► Gmail: "Success" Email
```

> 💡 **কীভাবে Error হবে?** Supabase Errors Table এ `email` Column এ Unique Constraint দেওয়া থাকবে। একই Email দুবার Insert করলে Error হবে।

---

## 📋 Google Sheets সেটআপ

### ✅ ধাপ ১ — Google Sheets তৈরি করো

- **"Error Handling"** নামে একটি নতুন Google Sheet তৈরি করো
- নিচের Columns ও Data দাও:

| Name | Email |
|------|-------|
| `Wasim` | `mdwasimu02@gmail.com` |

---

## ⚙️ n8n Workflow সেটআপ

### ✅ ধাপ ২ — Google Sheets Trigger যোগ করো

- **Nodes Panel** ওপেন করো
- **"Google Sheets"** সার্চ করো
- **"On row added"** সিলেক্ট করো
- Value সেট করো
- **"Fetch Test Event"** বাটনে ক্লিক করো

> ✔️ Google Sheets এর Data n8n এ চলে আসবে।

---

## 🗄️ Supabase Table সেটআপ

### ✅ ধাপ ৩ — Errors Table তৈরি করো

- Supabase Dashboard এ যাও
- **Table Editor** এ ক্লিক করো
- **"New Table"** এ ক্লিক করো
- **Name** দাও: `Errors`
- নিচের Columns যোগ করো:

![Errors Table Setup](https://imgur.com/WpbRShr.png)

> ⚠️ **`email` Column এ Unique Constraint দাও** — এটাই Error তৈরির মূল কারণ। একই Email দুবার Insert হলে Supabase Error দেবে।

- **Save** বাটনে ক্লিক করো

---

### ✅ ধাপ ৪ — Supabase Node যোগ করো

- **Google Sheets Trigger** এর **`+`** বাটনে ক্লিক করো
- **"Supabase"** সার্চ করো → **"Create a row"** সিলেক্ট করো
- **Drag & Drop** করে নিচের Value সেট করো:
- **"Add Field"** এ ক্লিক করে Field যোগ করো

| Supabase Field | Google Sheets থেকে |
|----------------|--------------------|
| `name` | `{{ $json.Name }}` |
| `email` | `{{ $json.Email }}` |

- **"Execute step"** এ ক্লিক করো

> 🔴 **গুরুত্বপূর্ণ — On Error: Continue সেট করো**
> Create a row Node এর **Settings** Tab এ যাও → **"On Error"** Option টি **`Continue`** সিলেক্ট করো।
> এটা না করলে Error হলে Workflow সম্পূর্ণ বন্ধ হয়ে যাবে — IF Node পর্যন্ত পৌঁছাবে না।

```
Create a row → Settings Tab → On Error → Continue ✅
```

---

## 🔀 IF Condition সেটআপ

### ✅ ধাপ ৫ — IF Node যোগ করো

- **Create a row** এর **`+`** বাটনে ক্লিক করো
- **"if"** সার্চ করো এবং ক্লিক করো
- Condition সেট করো:

```
{{ $json.error }}   is not empty
```

| Condition Field | Value |
|-----------------|-------|
| **Value** | `{{ $json.error }}` |
| **Operation** | `is equal to` |

> 💡 Supabase Insert সফল হলে `$json.error` **ফাঁকা** থাকে। Error হলে `$json.error` এ **Error Message** আসে।

---

## ✉️ Gmail Nodes সেটআপ

### ✅ ধাপ ৬ — Success Email (False Branch)

- **IF** এর **False** এর **`+`** বাটনে ক্লিক করো
- **"Gmail"** সার্চ করো → **"Send a message"** সিলেক্ট করো
- Node এর **নাম পরিবর্তন** করো: `Success`
- নিচের মতো Value সেট করো:

| Field | Value |
|-------|-------|
| **To** | Admin Email |
| **Subject** | `✅ Data Successfully Inserted` |
| **Message** | Name ও Email সহ Success নোটিফিকেশন |

---

### ✅ ধাপ ৭ — Failed Email (True Branch)

- **IF** এর **True** এর **`+`** বাটনে ক্লিক করো
- **"Gmail"** সার্চ করো → **"Send a message"** সিলেক্ট করো
- Node এর **নাম পরিবর্তন** করো: `Failed`
- নিচের মতো Value সেট করো:

| Field | Value |
|-------|-------|
| **To** | Admin Email |
| **Subject** | `❌ Data Insert Failed — Duplicate Email` |
| **Message** | Error Message ও Email সহ Failed নোটিফিকেশন |

---

## ▶️ Workflow Test করো

### ✅ ধাপ ৮ — Execute ও যাচাই করো

Google Sheets এ Value যোগ করো এবং **Execute Workflow** বাটনে ক্লিক করো।

**দুটো পরিস্থিতি:**

| পরিস্থিতি | ফলাফল | Email |
|-----------|--------|-------|
| **নতুন Unique Email** | Supabase এ Insert সফল | ✅ **Success** Email যাবে |
| **একই Email আবার** | Supabase Error হবে | ❌ **Failed** Email যাবে |

---

## 📚 Error Handling কেন দরকার?

```
Error Handling ছাড়া:
Supabase Error হলে → Workflow বন্ধ হয়ে যায় → কিছুই জানা যায় না ❌

Error Handling সহ:
Supabase Error হলে → Error ধরা পড়ে → Failed Email যায় → সমস্যা জানা যায় ✅
```

### বাস্তব ব্যবহার:

| Error ধরন | উদাহরণ |
|-----------|---------|
| **Duplicate Entry** | একই Email দুবার Insert |
| **Missing Field** | Required Column এ কোনো Value নেই |
| **Connection Error** | Supabase Offline থাকলে |
| **Type Mismatch** | সংখ্যার জায়গায় Text দিলে |

---

## 📋 Quick Reference

| ধাপ | কাজ | কোথায় |
|-----|-----|--------|
| ১ | Google Sheets তৈরি ও Data দাও | Google Sheets |
| ২ | Google Sheets Trigger সেটআপ | n8n |
| ৩ | Errors Table তৈরি (Unique Email) | Supabase |
| ৪ | Supabase Create a row সেটআপ | n8n |
| ৫ | IF Condition — `$json.error` চেক | n8n |
| ৬ | Success Gmail (False Branch) | n8n |
| ৭ | Failed Gmail (True Branch) | n8n |
| ৮ | Execute ও Test করো | n8n |

---

*Error Handling সেটআপ করলে Workflow কখনো নীরবে ব্যর্থ হয় না — সবসময় জানা যাবে কী হয়েছে।*
