# 🧾 SAWA POS v 1.0
## Point of Sale & Restaurant/Cafe Operations Management System

> **SAWA POS** هو نظام نقاط بيع وإدارة تشغيل لبيئات المطاعم والمقاهي، مبني كتطبيق Windows Desktop باستخدام C# و.NET Windows Forms وSQL Server، ويغطي دورة التشغيل اليومية من تسجيل الدخول وفتح الوردية، مرورًا بالبيع والدفع والخصومات والضرائب والمصروفات والمرتجعات، وانتهاءً بإغلاق الوردية والتقارير ولوحة التحكم.

---

## 📌 نظرة عامة

SAWA POS ليس مجرد شاشة كاشير لإدخال منتج وحساب سعر. الفكرة الأساسية في النظام هي بناء **دورة تشغيل مترابطة** بحيث ترتبط العمليات المالية والتشغيلية بالمستخدم والوردية والطلب وطريقة الدفع.

```text
Login
  │
  ▼
Main Form
  │
  ├── Products / Categories
  ├── Users / Permissions
  ├── Settings
  └── Dashboard
  │
  ▼
Open Shift
  │
  ▼
POS
  │
  ├── Product
  ├── Variant
  ├── Quantity
  ├── Notes
  └── Discount
  │
  ▼
Order
  │
  ▼
Payment
  │
  ├── Cash
  ├── Visa/Card
  └── Split Cash + Visa
  │
  ▼
Completed Order
  │
  ├── Receipt
  ├── Reports
  └── Dashboard
  │
  ├── Refund
  └── Shift Closing
          │
          ├── Expected Cash
          ├── Actual Cash
          └── Difference
```
---

## 📚 جدول المحتويات

1. [نظرة عامة](#نظرة-عامة)
2. [التقنيات المستخدمة](#التقنيات-المستخدمة)
3. [المعمارية البرمجية](#المعمارية-البرمجية)
4. [هيكل المشروع](#هيكل-المشروع)
5. [لقطات الواجهة (UI Showcase)](#لقطات-الواجهة-ui-showcase)
6. [العرض المرئي الكامل](#العرض-المرئي-الكامل)
7. [تسجيل الدخول والمستخدمون](#تسجيل-الدخول-والمستخدمون)
8. [الأدوار والصلاحيات](#الأدوار-والصلاحيات)
9. [إدارة التصنيفات](#إدارة-التصنيفات)
10. [إدارة المنتجات](#إدارة-المنتجات)
11. [Variants / Sizes](#variants--sizes)
12. [دورة حياة الطلب](#دورة-حياة-الطلب)
13. [الطلبات المعلقة](#الطلبات-المعلقة)
14. [شاشة POS](#شاشة-pos)
15. [الخصومات](#الخصومات)
16. [VAT والضرائب](#vat-والضرائب)
17. [الدفع](#الدفع)
18. [الفواتير والطباعة](#الفواتير-والطباعة)
19. [المرتجعات (Refunds)](#المرتجعات-refunds)
20. [إدارة الورديات](#إدارة-الورديات)
21. [حساب Expected Cash](#حساب-expected-cash)
22. [إغلاق الوردية](#إغلاق-الوردية)
23. [Blind Close](#blind-close)
24. [Shift Corrections](#shift-corrections)
25. [المصروفات](#المصروفات)
26. [Dashboard](#dashboard)
27. [التقارير](#التقارير)
28. [PDF والبريد الإلكتروني](#pdf-والبريد-الإلكتروني)
29. [الإعدادات](#الإعدادات)
30. [تصميم قاعدة البيانات](#تصميم-قاعدة-البيانات)
31. [ERD](#erd)
32. [سلامة البيانات والمعاملات](#سلامة-البيانات-والمعاملات)
33. [معالجة الأخطاء](#معالجة-الأخطاء)
34. [ملاحظات تقنية وأمنية](#ملاحظات-تقنية-وأمنية)
35. [نقاط يجب معرفتها عن Schema الحالي](#نقاط-يجب-معرفتها-عن-schema-الحالي)
36. [حدود نطاق الإصدار الحالي](#حدود-نطاق-الإصدار-الحالي)
37. [دورة حياة النظام الكاملة](#دورة-حياة-النظام-الكاملة)
38. [الخلاصة](#الخلاصة)

- **C#**
- **.NET / Windows Forms**
- **SQL Server**
- **ADO.NET / Microsoft.Data.SqlClient**
- **T-SQL**
- **3-Tier Architecture**
- **Object-Oriented Programming (OOP)**
- **Visual Studio**
- **Git / GitHub**
- **Draw.io** لتصميم ERD
- **iTextSharp** لتوليد PDF
- **QRCoder** لتوليد QR Code
- **Brevo API** لإرسال التقارير بالبريد الإلكتروني
- **Inno Setup** لحزمة التثبيت

---

# 🏗️ المعمارية البرمجية

النظام مبني وفق **3-Tier Architecture**:

```text
┌───────────────────────────────┐
│      Presentation Layer       │
│     Windows Forms / UI        │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│        Business Layer         │
│  Business Rules & Calculations│
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       Data Access Layer       │
│       SQL / ADO.NET           │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│          SQL Server           │
└───────────────────────────────┘
```

## Presentation Layer

مسؤولة عن:

- Forms وUserControls.
- إدخال البيانات.
- عرض البيانات.
- التنقل بين أجزاء النظام.
- التحقق الأولي من المدخلات.
- عرض رسائل النجاح والأخطاء.
- تطبيق صلاحيات الوصول على الواجهات.

## Business Layer

مسؤولة عن:

- قواعد العمل.
- حساب الإجماليات.
- الخصومات.
- VAT.
- حالات الطلبات.
- حالات الورديات.
- عمليات الدفع.
- عمليات المرتجعات.
- حساب Expected Cash / Difference.
- التحقق من العمليات قبل تنفيذها.

## Data Access Layer

مسؤولة عن:

- الاتصال بـSQL Server.
- `SELECT / INSERT / UPDATE / DELETE`.
- `SqlConnection`.
- `SqlCommand`.
- `SqlDataReader`.
- `DataTable`.
- SQL Parameters.
- Transactions للعمليات متعددة الخطوات.

---

# 📁 هيكل المشروع

المشروع مقسم إلى ثلاثة أجزاء رئيسية:

```text
SAWA POS
│
├── BusinessLayer
│   ├── Users
│   ├── Security
│   ├── Products
│   ├── Categories
│   ├── Orders
│   ├── Payments
│   ├── Expenses
│   ├── Shifts
│   ├── Shift Corrections
│   ├── Reports
│   └── Settings
│
├── DataAccessLayer
│   ├── Users
│   ├── People
│   ├── Roles
│   ├── Products
│   ├── ProductVariants
│   ├── Categories
│   ├── Orders
│   ├── Payments
│   ├── Expenses
│   ├── Shifts
│   ├── ShiftCorrections
│   ├── Reports
│   └── Settings
│
└── Resturant & Cafe POS System
    ├── Login
    ├── Main Form
    ├── POS
    ├── Products
    ├── Categories
    ├── Orders
    ├── Payment
    ├── Refund
    ├── Shifts
    ├── Expenses
    ├── Users
    ├── Settings
    ├── Dashboard
    └── Reports
```

---

# 🖥️ لقطات الواجهة (UI Showcase)

| الشاشة | Preview |
|---|---|
| 🏠 Main Dashboard | ![Main Dashboard](Images/Sawa_Screen%20(9).jpg) |
| 🧾 POS Screen | ![POS Screen](Images/Sawa_Screen%20(10).png) |
| 🍔 Product Details | ![Product Details](Images/Sawa_Screen%20(11).png) |
| 📊 Sales Summary | ![Sales Summary](Images/Sawa_Screen%20(32).png) |
| 📋 Orders Report | ![Orders Report](Images/Sawa_Screen%20(1).png) |

> 📁 يحتوي المشروع على معرض صور إضافي داخل مجلد [`Images`](./Images) لواجهات النظام المختلفة، بما في ذلك المستخدمين والورديات والمصروفات والإعدادات والتقارير.

---

# 🎥 العرض المرئي الكامل

يتوفر تسجيل مرئي يستعرض دورة تشغيل النظام عمليًا، ويغطي:

- 🔐 تسجيل الدخول والصلاحيات.
- 💵 فتح الوردية والنقد الافتتاحي.
- 🧾 تنفيذ البيع من شاشة POS.
- 🍔 اختيار المنتجات والـVariants.
- 📝 إضافة الملاحظات.
- 💸 تطبيق الخصومات.
- 💳 الدفع النقدي والبطاقة والدفع المجزأ.
- ⏸️ تعليق الطلبات واستكمالها.
- 💰 تسجيل المصروفات.
- ↩️ تنفيذ المرتجعات.
- 🔒 إغلاق الوردية.
- 👀 Blind Close.
- 📊 Dashboard.
- 📈 Reports.
- 📄 PDF Export.

### ▶️ مشاهدة الفيديو

[🎬 اضغط هنا لمشاهدة العرض المرئي الكامل للنظام على Google Drive](https://drive.google.com/file/d/1bH1sfMtRQG6BdZno-1JVCAXDOcEUlPZo/view?usp=sharing)

---

# 🔐 تسجيل الدخول والمستخدمون

النظام يفصل بين بيانات الشخص وبيانات حساب الدخول:

```text
Person
   │
   ▼
User
   │
   ▼
Role
```

## People

يحتوي على:

- `PersonID`
- `FirstName`
- `LastName`
- `Phone`
- `Address`

## Users

يحتوي على:

- `UserID`
- `PersonID`
- `RoleID`
- `Username`
- `HashedPassword`
- `IsActive`
- `Permissions`

---

# 🛡️ الأدوار والصلاحيات

التطبيق الحالي يستخدم دورين أساسيين:

```text
Admin
Cashier
```

إلى جانب Permissions مخزنة كـBit Flags.

الصلاحيات تشمل:

```text
Dashboard
POS
Orders
Products
Categories
Expenses
Reports
Users
Shifts
Settings
```

التدفق:

```text
User clicks feature
        │
        ▼
Permission Check
        │
   ┌────┴────┐
   │         │
Allowed    Denied
   │         │
   ▼         ▼
Continue   Message
```

> ⚠️ جدول `Roles` يسمح نظريًا بأدوار إضافية، لكن منطق التطبيق الحالي مبني على Admin وCashier؛ لذلك لا ينبغي وصف النظام بأنه Role Management ديناميكي كامل.

---

# 🗂️ إدارة التصنيفات

كل منتج يرتبط بتصنيف واحد:

```text
Category
   │
   └── Products
```

تدعم شاشة Categories:

- إضافة التصنيف.
- تعديل التصنيف.
- تفعيل/تعطيل التصنيف.
- وصف التصنيف.
- صورة التصنيف.
- البحث والتصفية.

الحقول:

```text
CategoryID
CategoryName
IsActive
Description
CreatedAt
ImagePath
```

---

# 🍔 إدارة المنتجات

المنتج هو الوحدة الأساسية في شاشة POS.

```text
Product
├── ProductID
├── CategoryID
├── ProductName
├── Description
├── Price
├── IsAvailable
└── ImagePath
```

الوظائف:

- إضافة منتج.
- تعديل المنتج.
- تغيير السعر.
- تغيير التصنيف.
- تغيير حالة التوفر.
- إضافة صورة.
- البحث.
- التصفية.
- تعطيل المنتج دون حذف السجل.

---

# 📏 Variants / Sizes

يدعم النظام **Product Variants / Sizes**.

مثال:

```text
Burger
│
├── Small   → Price A
├── Medium  → Price B
└── Large   → Price C
```

التصميم:

```text
Products
    │
    ▼
ProductVariants
    │
    ▼
Variants
```

`ProductVariants` يحدد:

- المنتج.
- الـVariant.
- سعر هذا الـVariant لهذا المنتج.

> ⚠️ لا يوجد في الإصدار الحالي كيان مستقل لـAdd-ons مثل `AddOns` أو `OrderItemAddOns`. لذلك الوصف الصحيح للميزة هو **Variants / Sizes** وليس Add-ons.

---

# 🧾 دورة حياة الطلب

```text
POS
 │
 ▼
Create Order
 │
 ├── Product
 ├── Variant
 ├── Quantity
 ├── Notes
 └── Discount
 │
 ▼
Calculate
 │
 ├── SubTotal
 ├── Discount
 ├── VAT
 └── Total
 │
 ▼
Payment
 │
 ▼
Completed
```

كل طلب مرتبط بـ:

- الوردية.
- المستخدم.
- عناصر الطلب.
- عمليات الدفع.

---

## 🧩 Order Items

الطلب لا يخزن المنتجات داخله مباشرة، وإنما من خلال `OrderItems`.

```text
Order
 │
 └── OrderItems
       │
       ├── Product
       ├── Variant
       ├── Quantity
       ├── UnitPrice
       ├── Discount
       ├── TotalPrice
       └── Notes
```

أهم قرار تصميمي هنا هو **Historical Pricing**.

```text
Product Price at Sale = 2.000
           │
           ▼
OrderItem.UnitPrice = 2.000
           │
           ▼
Product Price later = 2.500

Old Order still = 2.000
```

وبذلك لا تتغير الفواتير التاريخية عند تعديل أسعار المنتجات.

---

# ⏸️ الطلبات المعلقة

يمكن أن يبقى الطلب في حالة Pending قبل إتمام الدفع.

```text
Create Order
     ↓
Add Items
     ↓
Pending
     ↓
Orders Screen
     ↓
Resume
     ↓
Payment
     ↓
Completed
```

وهذا يسمح للكاشير بالتعامل مع أكثر من عميل دون فقدان الطلب الحالي.

---

# 🧮 شاشة POS

شاشة POS هي الواجهة التشغيلية الأساسية للكاشير.

تجمع:

- Categories.
- Products.
- Variants.
- Quantity.
- Notes.
- Discounts.
- SubTotal.
- VAT.
- Total.
- Payment.

التدفق:

```text
Category
   ↓
Product
   ↓
Variant
   ↓
Quantity
   ↓
Notes
   ↓
Discount
   ↓
Totals
   ↓
Payment
```

---

# 💸 الخصومات

يدعم النظام:

- خصم على مستوى Order.
- خصم على مستوى OrderItem.
- Max Discount Percentage.
- Enable/Disable Discount.

```text
Original Price
      ↓
Discount
      ↓
Discounted Value
      ↓
Tax Calculation
      ↓
Final Total
```

---

# 🧮 VAT والضرائب

يمكن تفعيل أو تعطيل VAT من الإعدادات.

يحتفظ الطلب نفسه بمعلومات الضريبة الخاصة به:

```text
VATPercentage
TaxNumber
IsTaxInclusive
```

## Inclusive

السعر يتضمن الضريبة.

```text
Gross
 ↓
Extract VAT
 ↓
Net
```

الحساب المفاهيمي:

```text
VAT = Total - (Total / (1 + VAT%))
```

## Exclusive

السعر لا يتضمن الضريبة.

```text
Net
 ↓
VAT
 ↓
Net + VAT
```

إعدادات VAT تشمل:

- `VAT`
- `VatEnable`
- `TaxMethod`
- `TaxNumber`
- `IsTaxInclusive`
- `DecimalPlaces`

---

# 💳 الدفع

وسائل الدفع الفعلية في التطبيق:

```text
Cash = 1
Visa/Card = 2
```

## Cash

```text
Total
 ↓
Paid Amount
 ↓
Change
```

## Card

```text
Total
 ↓
Card Payment
```

## Split Payment

يمكن تقسيم المبلغ:

```text
Order Total = 10.000

Cash = 4.000
Card = 6.000

4 + 6 = 10
```

وتسجل كل وسيلة دفع كسجل مستقل في `Payments`.

---

# 🖨️ الفواتير والطباعة

بعد نجاح الدفع:

```text
Order
 ↓
Payment
 ↓
Completed
 ↓
Receipt
```

يدعم النظام إعدادات:

- Restaurant Name.
- Logo.
- Phone.
- Address.
- Currency.
- Decimal Places.
- Tax Number.
- Printer Name.
- Auto Print.
- QR Code.

ويحتوي المشروع على منطق طباعة وفاتورة يعتمد على إعدادات المنشأة.

---

# ↩️ المرتجعات (Refunds)

المرتجع لا يحذف الطلب الأصلي.

بدلًا من ذلك:

```text
Original Order
      │
      ▼
Refund Order
      │
      ▼
RefundTracking
```

يحفظ `RefundTracking`:

- `TrackingID`
- `RefundOrderID`
- `OriginalOrderID`
- `RefundReason`
- `RefundDate`

## Partial Refund

مثال:

```text
Original:
Burger × 2
Fries  × 1

Refund:
Burger × 1
```

يحاول منطق النظام منع إرجاع كمية تتجاوز الكمية المتاحة للإرجاع.

## Full Refund

يمكن إرجاع الطلب كاملًا عندما تكون الكمية والقيمة مؤهلة لذلك.

## الأثر المالي

يتم تمثيل المرتجع كعملية مالية سالبة في النموذج المستخدم في التطبيق:

```text
Refund Order
   ↓
Negative Order Values
   ↓
Negative Payment
```

وبذلك يمكن للتقارير حساب الأثر الصافي للمرتجعات.

---

# 🕒 إدارة الورديات

الوردية هي محور الرقابة النقدية.

```text
Open Shift
    ↓
Orders
    ↓
Payments
    ↓
Expenses
    ↓
Refunds
    ↓
Close Shift
```

بيانات الوردية:

```text
ShiftID
OpenedByUser
OpenedAt
ClosedAt
OpeningCash
Status
ExpectedCash
ActualCash
Difference
Notes
```

---

# 💰 حساب Expected Cash

الصيغة التنفيذية الأدق تعتمد على مدفوعات النقد الفعلية:

```text
Expected Cash
=
Opening Cash
+
Cash Payment Amounts
-
Cash Expenses
```

وبما أن المرتجعات النقدية تسجل كقيم سالبة في النموذج المالي، فهي تؤثر تلقائيًا على صافي النقد.

---

# 🔒 إغلاق الوردية

عند الإغلاق:

```text
Expected Cash
      │
      ▼
Actual Cash
      │
      ▼
Difference
```

الحساب:

```text
Difference = Actual Cash - Expected Cash
```

النتائج:

```text
0     → No Difference
< 0   → Shortage
> 0   → Overage
```

---

# 👀 Blind Close

يوجد إعداد:

```text
EnableBlindClose
```

عند تفعيله يمكن للكاشير إدخال `ActualCash` دون عرض `ExpectedCash` له مسبقًا.

---

# 🛠️ Shift Corrections

يوجد كيان مستقل لتصحيحات الوردية:

```text
ShiftCorrections
```

ويخزن:

- القيمة القديمة.
- القيمة الجديدة.
- نوع التصحيح.
- السبب.
- المستخدم.
- التاريخ.

ويستخدم لتصحيحات مثل:

```text
Opening Cash Correction
Actual Cash Correction
```

> ملاحظة: هذا ليس نظام Audit Log عامًا؛ هو سجل تصحيحات مالي متخصص بالوردية.

---

# 💵 المصروفات

المصروف مرتبط بالمستخدم والوردية:

```text
User
  │
  ▼
Expense
  │
  ▼
Shift
```

بيانات المصروف:

```text
ExpenseID
CreatedByUserID
ShiftID
Amount
Reason
CreatedAt
PaymentSource
Description
StatusExpense
```

المصروف النقدي يؤثر على Expected Cash.

---

# 📊 Dashboard

لوحة التحكم تجمع المؤشرات التشغيلية والمالية:

```text
Dashboard
│
├── KPIs
├── Current Shift
├── Alerts
├── Hourly Sales
├── Category Sales
├── Top Products
└── Quick Actions
```

وتتضمن مؤشرات مثل:

- Gross Sales.
- Net Sales.
- Total Orders.
- Average Order Value.
- Refunds.
- Discounts.
- VAT.
- Cash Collected.
- Card Collected.
- Shift Information.

---

# 📈 التقارير

يحتوي النظام على تقارير متخصصة، منها:

### 🧾 Sales Analysis

تحليل المبيعات.

### 📊 Sales Summary

ملخص المبيعات.

### 📋 Orders Report

تحليل الطلبات حسب التاريخ والحالة والمستخدم والوردية والقيمة.

### 💳 Payment Report

تحليل المدفوعات حسب Cash/Card.

### 💵 Expense Report

تحليل المصروفات.

### ↩️ Refund Report

تحليل عمليات المرتجعات.

### 🕒 Shift Report

تحليل الورديات والنقد المتوقع والفعلي والفروقات.

---

# 📄 PDF والبريد الإلكتروني

يدعم النظام تصدير التقارير إلى PDF باستخدام iTextSharp.

كما توجد شاشة لإرسال التقرير بالبريد الإلكتروني، ويستخدم النظام Brevo API.

```text
Report
  ↓
Generate
  ↓
PDF
  ↓
Email
  ↓
Brevo
```

---

# ⚙️ الإعدادات

`SystemSettings` هو مركز إعدادات المنشأة.

## معلومات المنشأة

- Restaurant Name.
- Logo.
- Phone.
- Address.
- Email.
- CR Number.
- Tax Number.

## المالية والضريبة

- Currency.
- VAT.
- VAT Enable.
- Tax Method.
- Decimal Places.
- Max Discount.

## الطباعة

- Printer Name.
- Auto Print Receipt.
- Show QR Code.

## التشغيل

- Blind Close.
- Enable Discount.
- Language.

## البريد

- Sender Email.
- Encrypted Brevo Key.

---

# 🗄️ تصميم قاعدة البيانات

قاعدة البيانات الحالية تتكون من الكيانات الرئيسية التالية:

```text
People
Users
Roles

Categories
Products
Variants
ProductVariants

Shifts
ShiftCorrections
Expenses

Orders
OrderItems
Payments
RefundTracking

SystemSettings
```

العلاقات الرئيسية:

```text
People
   │
   └── Users
          │
          ├── Roles
          ├── Shifts
          ├── Orders
          ├── Expenses
          └── ShiftCorrections

Categories
   │
   └── Products
          │
          └── ProductVariants
                  │
                  └── Variants

Shifts
   ├── Orders
   ├── Expenses
   └── ShiftCorrections

Orders
   ├── OrderItems
   └── Payments

RefundTracking
   └── Original Order ↔ Refund Order
```

📚 **وثيقة تصميم قاعدة البيانات التفصيلية:**  
[SAWA POS Database Design](./DATABASE_DESIGN.md)

---

# 🔗 ERD

تم تصميم قاعدة البيانات باستخدام نموذج علائقي، ويمكن الاحتفاظ بملف ERD عالي الدقة داخل المستودع.

![SAWA POS Database Schema](SawaPOS_Schema.drawio.png)

📥 [عرض ملف ERD بصيغة PDF](SawaPOS_Schema.drawio.pdf)

🔗 [مستودع تصميم قاعدة البيانات](https://github.com/zuhairSh/SAWA-POS-Database-Design)

---

# 🔄 سلامة البيانات والمعاملات

من أهم خصائص التصميم:

- SQL Parameters.
- Transactions للعمليات المالية المركبة.
- Historical Pricing.
- Foreign Key Relationships.
- Business Validation.
- ربط الطلب بالمستخدم والوردية.
- ربط المصروف بالوردية والمستخدم.
- ربط الدفع بالطلب.
- منع بعض العمليات بناءً على حالة الوردية.

## Transaction Pattern

```text
BEGIN TRANSACTION
       │
       ├── Operation 1
       ├── Operation 2
       ├── Operation 3
       │
       ▼
    COMMIT
```

وفي حالة الفشل:

```text
Failure
   ↓
ROLLBACK
```

يستخدم هذا النمط في العمليات التي تحتاج نجاحًا ذريًا عبر أكثر من جدول، مثل إنشاء الطلب مع عناصره، الدفع، والمرتجعات.

---

# 🧯 معالجة الأخطاء

تستخدم طبقة الوصول إلى البيانات نمط `try/catch` لمعالجة أخطاء الاتصال وتنفيذ الاستعلامات.

كما توجد آلية مركزية لمعالجة أخطاء قاعدة البيانات بدل ترك الاستثناءات تتسبب في انهيار غير متحكم به للواجهة.

---

# 🔐 ملاحظات تقنية وأمنية

## كلمات المرور

يتم تخزين كلمة المرور بصيغة Hash بدل Plain Text.

التنفيذ الحالي يعتمد على SHA-256. وللأنظمة الإنتاجية الحديثة يفضل استخدام Password KDF مخصص مثل:

- Argon2
- bcrypt
- PBKDF2
- scrypt

مع Salt مناسب.

## Remember Me

يستخدم التطبيق Windows DPAPI لحماية بيانات الاعتماد المحلية المرتبطة بالمستخدم.

## Connection String

يجب عدم وضع بيانات اعتماد SQL Server الحقيقية داخل Repository عام.

الأفضل استخدام:

```text
Environment Variables
User Secrets
Secure Secret Store
Encrypted Configuration
```

## Brevo API Key

يجب أيضًا حماية مفاتيح API وعدم نشر أسرار الإنتاج داخل Git.

---

# ⚠️ نقاط يجب معرفتها عن Schema الحالي

هناك بعض العلاقات التي يستخدمها التطبيق منطقيًا ولكنها ليست كلها ممثلة كـForeign Keys في قائمة القيود المقدمة.

أهم الأمثلة:

```text
Payments.OrderID
Payments.ShiftID

RefundTracking.RefundOrderID
RefundTracking.OriginalOrderID

OrderItems.VariantID
```

لذلك يوصى بمراجعة هذه القيود في SQL Server لضمان أن سلامة البيانات لا تعتمد على Business Layer وحدها.

---

# 🚫 حدود نطاق الإصدار الحالي

للدقة، الإصدار الحالي لا ينبغي وصفه على أنه:

- Inventory Management System.
- Purchasing System.
- Supplier Management System.
- Stock Movement System.
- Add-on Management System.

لا توجد في الـSchema والكود المقدم كيانات مستقلة لـ:

```text
Inventory
Purchases
PurchaseItems
Suppliers
StockMovements
AddOns
OrderItemAddOns
```

أما **Variants/Sizes** فهي موجودة فعليًا ومتكاملة مع المنتجات.

---

# 🧭 دورة حياة النظام الكاملة

```text
                    ┌──────────────┐
                    │    Login     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Main Form   │
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
      Products          Users           Settings
          │
          └────────────────┐
                           ▼
                    ┌──────────────┐
                    │  Open Shift  │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │     POS      │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    Order     │
                    └──────┬───────┘
                           │
                    ┌──────┴───────┐
                    │              │
                 Pending        Payment
                    │              │
                    │       ┌──────┼──────┐
                    │       │      │      │
                    │      Cash   Card  Split
                    │       │      │      │
                    └───────┴──────┴──────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Completed   │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
           Receipt      Dashboard     Reports
              │
              ▼
           Possible
            Refund
              │
              ▼
        ┌───────────────┐
        │ Shift Closing │
        └───────┬───────┘
                │
        ┌───────┼────────┐
        │       │        │
        ▼       ▼        ▼
     Expected Actual  Difference
       Cash    Cash
```

---

# 💡 الخلاصة

SAWA POS هو نظام POS مكتمل نسبيًا من ناحية دورة التشغيل اليومية للمطاعم والمقاهي، ويجمع بين:

```text
👤 Users
🔐 Permissions
🗂️ Categories
🍔 Products
📏 Variants
🧾 Orders
💳 Payments
↩️ Refunds
🕒 Shifts
💵 Expenses
📊 Dashboard
📈 Reports
🖨️ Printing
🧮 VAT
📄 PDF
📧 Email
⚙️ Settings
```

القيمة الأساسية في التصميم هي أن عملية البيع ليست معزولة، بل مرتبطة بالمستخدم والوردية والطلب والدفع والتقارير، مع وجود Transactional Operations للعمليات المالية المركبة وحفظ السعر التاريخي للطلبات.

