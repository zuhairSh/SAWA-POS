# 🧾 SAWA POS v1.0
## Point of Sale & Restaurant/Cafe Operations Management System

> **SAWA POS** هو نظام نقاط بيع وإدارة تشغيل لبيئات المطاعم والمقاهي، مبني كتطبيق Windows Desktop باستخدام C# و.NET Windows Forms وSQL Server، ويغطي دورة التشغيل اليومية من تسجيل الدخول وفتح الوردية، مرورًا بالبيع والدفع والخصومات والضرائب والمصروفات والمرتجعات، وانتهاءً بإغلاق الوردية والتقارير ولوحة التحكم.

---

<a id="toc"></a>

# 📚 جدول المحتويات

- [نظرة عامة](#overview)
- [التقنيات المستخدمة](#tech)
- [المعمارية البرمجية](#architecture)
- [هيكل المشروع](#structure)
- [لقطات الواجهة (UI Showcase)](#ui)
- [العرض المرئي الكامل](#demo)
- [تسجيل الدخول والمستخدمون](#auth)
- [الأدوار والصلاحيات](#roles)
- [إدارة التصنيفات](#categories)
- [إدارة المنتجات](#products)
- [Variants / Sizes](#variants)
- [دورة حياة الطلب](#order-lifecycle)
- [الطلبات المعلقة](#pending)
- [شاشة POS](#pos)
- [الخصومات](#discounts)
- [VAT والضرائب](#vat)
- [الدفع](#payment)
- [الفواتير والطباعة](#receipts)
- [المرتجعات (Refunds)](#refunds)
- [إدارة الورديات](#shifts)
- [حساب Expected Cash](#expected-cash)
- [إغلاق الوردية](#shift-closing)
- [Blind Close](#blind-close)
- [Shift Corrections](#shift-corrections)
- [المصروفات](#expenses)
- [Dashboard](#dashboard)
- [التقارير](#reports)
- [PDF والبريد الإلكتروني](#pdf-email)
- [الإعدادات](#settings)
- [تصميم قاعدة البيانات](#database)
- [ERD](#erd)
- [سلامة البيانات والمعاملات](#integrity)
- [معالجة الأخطاء](#errors)
- [دورة حياة النظام الكاملة](#system-lifecycle)
- [الخلاصة](#summary)

---

<a id="overview"></a>

# 📌 نظرة عامة

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

### ✨ أبرز النقاط التقنية

| الجانب | ما تم تنفيذه |
|---|---|
| المعمارية | 3-Tier Architecture مع فصل واضح بين الواجهة والمنطق والبيانات |
| البيانات | قاعدة SQL Server علائقية بـ 15 جدولًا وعلاقات Foreign Key واضحة |
| الأمان | كلمات مرور مجزّأة (Hashed) وصلاحيات تفصيلية لكل وحدة |
| الدقة المالية | Historical Pricing وTransactions للعمليات المركبة |
| التشغيل | ورديات، دفع مجزأ، مرتجعات، Blind Close، تصحيحات مالية |
| المخرجات | Dashboard وتقارير وPDF وإرسال بالبريد |

---

<a id="tech"></a>

# 🛠️ التقنيات المستخدمة

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

<a id="architecture"></a>

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

- Forms وUserControls.
- إدخال البيانات وعرضها والتنقل بين أجزاء النظام.
- التحقق الأولي من المدخلات.
- عرض رسائل النجاح والأخطاء.
- تطبيق صلاحيات الوصول على الواجهات.

## Business Layer

- قواعد العمل وحساب الإجماليات.
- الخصومات وVAT.
- حالات الطلبات والورديات.
- عمليات الدفع والمرتجعات.
- حساب Expected Cash / Difference.
- التحقق من العمليات قبل تنفيذها.

## Data Access Layer

- الاتصال بـSQL Server عبر `SqlConnection` و`SqlCommand` و`SqlDataReader` و`DataTable`.
- عمليات `SELECT / INSERT / UPDATE / DELETE` باستخدام SQL Parameters.
- Transactions للعمليات متعددة الخطوات.

---

<a id="structure"></a>

# 📁 هيكل المشروع

المشروع مقسم إلى ثلاثة أجزاء رئيسية:

```text
SAWA POS
│
├── BusinessLayer          → قواعد العمل لكل وحدة
├── DataAccessLayer        → الوصول إلى قاعدة البيانات
└── Restaurant & Cafe POS System   → الواجهة (Windows Forms)
```

وتغطي الوحدات: المستخدمين والصلاحيات، المنتجات والتصنيفات، الطلبات والمدفوعات والمرتجعات، الورديات والتصحيحات، المصروفات، التقارير، الإعدادات.

---

<a id="ui"></a>

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

<a id="demo"></a>

# 🎥 العرض المرئي الكامل

يتوفر تسجيل مرئي يستعرض دورة تشغيل النظام عمليًا، ويغطي:

- 🔐 تسجيل الدخول والصلاحيات.
- 💵 فتح الوردية والنقد الافتتاحي.
- 🧾 تنفيذ البيع من شاشة POS.
- 🍔 اختيار المنتجات والـVariants وإضافة الملاحظات.
- 💸 تطبيق الخصومات.
- 💳 الدفع النقدي والبطاقة والدفع المجزأ.
- ⏸️ تعليق الطلبات واستكمالها.
- 💰 تسجيل المصروفات.
- ↩️ تنفيذ المرتجعات.
- 🔒 إغلاق الوردية وBlind Close.
- 📊 Dashboard والتقارير وتصدير PDF.

### ▶️ مشاهدة الفيديو

[🎬 اضغط هنا لمشاهدة العرض المرئي الكامل للنظام على Google Drive](https://drive.google.com/file/d/1bH1sfMtRQG6BdZno-1JVCAXDOcEUlPZo/view?usp=sharing)

---

<a id="auth"></a>

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

- **People:** البيانات الشخصية (الاسم، الهاتف، العنوان).
- **Users:** حساب الدخول المرتبط بالشخص وبدوره، مع كلمة مرور مجزّأة (Hashed) وحالة تفعيل للحساب وصلاحيات مخصصة.

---

<a id="roles"></a>

# 🛡️ الأدوار والصلاحيات

التطبيق الحالي يستخدم دورين أساسيين:

```text
Admin
Cashier
```

إلى جانب صلاحيات تفصيلية لكل وحدة، تشمل:

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

<a id="categories"></a>

# 🗂️ إدارة التصنيفات

كل منتج يرتبط بتصنيف واحد:

```text
Category
   │
   └── Products
```

تدعم شاشة Categories:

- إضافة التصنيف وتعديله.
- تفعيل/تعطيل التصنيف.
- وصف التصنيف وصورته.
- البحث والتصفية.

---

<a id="products"></a>

# 🍔 إدارة المنتجات

المنتج هو الوحدة الأساسية في شاشة POS.

الوظائف:

- إضافة منتج وتعديله.
- تغيير السعر والتصنيف وحالة التوفر.
- إضافة صورة.
- البحث والتصفية.
- تعطيل المنتج دون حذف السجل.

---

<a id="variants"></a>

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

`ProductVariants` يحدد المنتج والـVariant وسعر هذا الـVariant لهذا المنتج.

> ⚠️ لا يوجد في الإصدار الحالي كيان مستقل لـAdd-ons؛ لذلك الوصف الصحيح للميزة هو **Variants / Sizes** وليس Add-ons.

---

<a id="order-lifecycle"></a>

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

كل طلب مرتبط بالوردية والمستخدم وعناصر الطلب وعمليات الدفع.

## 🧩 Order Items

الطلب لا يخزن المنتجات داخله مباشرة، وإنما من خلال `OrderItems` (المنتج، الـVariant، الكمية، سعر الوحدة، الخصم، الملاحظات).

أهم قرار تصميمي هنا هو **Historical Pricing**:

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

<a id="pending"></a>

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

<a id="pos"></a>

# 🧮 شاشة POS

شاشة POS هي الواجهة التشغيلية الأساسية للكاشير، وتجمع التصنيفات والمنتجات والـVariants والكمية والملاحظات والخصومات والإجماليات والدفع.

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

<a id="discounts"></a>

# 💸 الخصومات

يدعم النظام:

- خصم على مستوى Order.
- خصم على مستوى OrderItem.
- حد أقصى لنسبة الخصم.
- تفعيل/تعطيل الخصومات من الإعدادات.

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

<a id="vat"></a>

# 🧮 VAT والضرائب

يمكن تفعيل أو تعطيل VAT من الإعدادات، ويحتفظ كل طلب بمعلومات الضريبة الخاصة به وقت إنشائه.

- **Inclusive:** السعر يتضمن الضريبة، فتُستخرج منه.
- **Exclusive:** السعر لا يتضمن الضريبة، فتُضاف عليه.

```text
Inclusive:  Gross → Extract VAT → Net
Exclusive:  Net → Add VAT → Net + VAT
```

كما يدعم النظام ضبط عدد الخانات العشرية وطريقة الضريبة من الإعدادات.

---

<a id="payment"></a>

# 💳 الدفع

وسائل الدفع الفعلية في التطبيق:

```text
Cash
Visa/Card
```

## Cash

```text
Total → Paid Amount → Change
```

## Card

```text
Total → Card Payment
```

## Split Payment

يمكن تقسيم المبلغ بين وسيلتين:

```text
Order Total = 10.000

Cash = 4.000
Card = 6.000

4 + 6 = 10
```

وتسجل كل وسيلة دفع كسجل مستقل في `Payments`.

---

<a id="receipts"></a>

# 🖨️ الفواتير والطباعة

بعد نجاح الدفع:

```text
Order → Payment → Completed → Receipt
```

يعتمد منطق الطباعة والفاتورة على إعدادات المنشأة، ويدعم:

- اسم المنشأة والشعار وبيانات التواصل.
- العملة وعدد الخانات العشرية.
- الرقم الضريبي.
- اختيار الطابعة والطباعة التلقائية.
- QR Code.

---

<a id="refunds"></a>

# ↩️ المرتجعات (Refunds)

المرتجع لا يحذف الطلب الأصلي، بل يُنشأ له طلب مرتجع مرتبط به:

```text
Original Order
      │
      ▼
Refund Order
      │
      ▼
RefundTracking
```

يحفظ `RefundTracking` الربط بين الطلب الأصلي وطلب المرتجع مع السبب والتاريخ.

## Partial Refund

```text
Original:
Burger × 2
Fries  × 1

Refund:
Burger × 1
```

يمنع منطق النظام إرجاع كمية تتجاوز الكمية المتاحة للإرجاع.

## Full Refund

يمكن إرجاع الطلب كاملًا عندما تكون الكمية والقيمة مؤهلة لذلك.

## الأثر المالي

يُمثَّل المرتجع كعملية مالية سالبة:

```text
Refund Order
   ↓
Negative Order Values
   ↓
Negative Payment
```

وبذلك تحسب التقارير الأثر الصافي للمرتجعات تلقائيًا.

---

<a id="shifts"></a>

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

تسجل الوردية: من فتحها ومتى، والنقد الافتتاحي، وحالتها، والنقد المتوقع والفعلي والفرق عند الإغلاق، وأي ملاحظات.

---

<a id="expected-cash"></a>

# 💰 حساب Expected Cash

يُحسب النقد المتوقع من النقد الافتتاحي والمدفوعات النقدية الفعلية، مطروحًا منها المصروفات النقدية:

```text
Expected Cash = Opening Cash + Cash Payments − Cash Expenses
```

وبما أن المرتجعات النقدية تسجل كقيم سالبة، فهي تؤثر تلقائيًا على صافي النقد.

---

<a id="shift-closing"></a>

# 🔒 إغلاق الوردية

```text
Expected Cash
      │
      ▼
Actual Cash
      │
      ▼
Difference
```

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

<a id="blind-close"></a>

# 👀 Blind Close

يوجد إعداد يفعّل **Blind Close**، وعند تفعيله يدخل الكاشير النقد الفعلي دون أن يرى النقد المتوقع مسبقًا، مما يقلل التأثر بالرقم المتوقع ويزيد دقة الجرد.

---

<a id="shift-corrections"></a>

# 🛠️ Shift Corrections

يوجد كيان مستقل لتصحيحات الوردية يخزن القيمة القديمة والجديدة ونوع التصحيح والسبب والمستخدم والتاريخ، ويستخدم لتصحيحات مثل:

```text
Opening Cash Correction
Actual Cash Correction
```

> ملاحظة: هذا ليس نظام Audit Log عامًا؛ هو سجل تصحيحات مالي متخصص بالوردية.

---

<a id="expenses"></a>

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

يسجل المصروف المبلغ والسبب ومصدر الدفع والوصف، والمصروف النقدي يؤثر على Expected Cash.

---

<a id="dashboard"></a>

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

- Gross Sales وNet Sales.
- Total Orders وAverage Order Value.
- Refunds وDiscounts وVAT.
- Cash Collected وCard Collected.
- معلومات الوردية الحالية.

---

<a id="reports"></a>

# 📈 التقارير

يحتوي النظام على تقارير متخصصة، منها:

- 🧾 **Sales Analysis:** تحليل المبيعات.
- 📊 **Sales Summary:** ملخص المبيعات.
- 📋 **Orders Report:** تحليل الطلبات حسب التاريخ والحالة والمستخدم والوردية والقيمة.
- 💳 **Payment Report:** تحليل المدفوعات حسب Cash/Card.
- 💵 **Expense Report:** تحليل المصروفات.
- ↩️ **Refund Report:** تحليل عمليات المرتجعات.
- 🕒 **Shift Report:** تحليل الورديات والنقد المتوقع والفعلي والفروقات.

---

<a id="pdf-email"></a>

# 📄 PDF والبريد الإلكتروني

يدعم النظام تصدير التقارير إلى PDF باستخدام iTextSharp، كما توجد شاشة لإرسال التقرير بالبريد الإلكتروني عبر Brevo API.

```text
Report → Generate → PDF → Email → Brevo
```

---

<a id="settings"></a>

# ⚙️ الإعدادات

يوجد مركز إعدادات للمنشأة يغطي:

- **معلومات المنشأة:** الاسم والشعار وبيانات التواصل.
- **المالية والضريبة:** العملة وVAT وطريقة الضريبة وعدد الخانات العشرية والحد الأقصى للخصم.
- **الطباعة:** الطابعة والطباعة التلقائية وQR Code.
- **التشغيل:** Blind Close وتفعيل الخصومات واللغة.
- **البريد:** إعدادات الإرسال.

---

<a id="database"></a>

# 🗄️ تصميم قاعدة البيانات

قاعدة البيانات الحالية تتكون من 15 جدولًا:

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

---

<a id="erd"></a>

# 🔗 ERD

تم تصميم قاعدة البيانات باستخدام نموذج علائقي، ويمكن الاطلاع على الـERD الكامل هنا:

![SAWA POS Database Schema](SawaPOS_Schema.png)

📥 [عرض ملف ERD بصيغة PDF](SawaPOS_Schema.drawio.pdf)

🔗 [مستودع تصميم قاعدة البيانات](https://github.com/zuhairSh/SAWA-POS-Database-Design)

---

<a id="integrity"></a>

# 🔄 سلامة البيانات والمعاملات

من أهم خصائص التصميم:

- SQL Parameters.
- Transactions للعمليات المالية المركبة.
- Historical Pricing.
- Foreign Key Relationships.
- Business Validation.
- ربط الطلب بالمستخدم والوردية، والمصروف بالوردية والمستخدم، والدفع بالطلب.
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

يستخدم هذا النمط في العمليات التي تحتاج نجاحًا ذريًا عبر أكثر من جدول، مثل إنشاء الطلب مع عناصره، والدفع، والمرتجعات.

---

<a id="errors"></a>

# 🧯 معالجة الأخطاء

تستخدم طبقة الوصول إلى البيانات نمط `try/catch` لمعالجة أخطاء الاتصال وتنفيذ الاستعلامات، كما توجد آلية مركزية لمعالجة أخطاء قاعدة البيانات بدل ترك الاستثناءات تتسبب في انهيار غير متحكم به للواجهة.

---

<a id="system-lifecycle"></a>

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

<a id="summary"></a>

# 💡 الخلاصة

**SAWA POS** هو نظام POS مكتمل في دورة التشغيل اليومية للمطاعم والمقاهي، ويجمع بين:

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

والحمد لله رب العالمين.
