# 🗂️ Repository Pattern & Unit Of Work (.NET 6) 

---

## 1️⃣ Repository Pattern Benefits

**Repository Pattern** هو Design Pattern بيعمل Layer وسيطة بين الـ Application والـ Data Source.

### أهم الفوائد:

* فصل منطق الوصول للبيانات عن الـ Business Logic
* كود أنضف وأسهل في الصيانة
* سهولة الاختبار (Unit Testing)
* تقليل الاعتماد المباشر على EF Core
<img width="1343" height="751" alt="image" src="https://github.com/user-attachments/assets/bd35a0cf-a5e5-4996-a2a2-a88b38517bff" />

🎯 الهدف: Clean Code + Maintainable Architecture.

---

## 2️⃣ Add Project Layers

بنقسّم المشروع إلى Layers:

* **API / Presentation Layer** → Controllers
* **Core / Domain Layer** → Entities & Interfaces
* **Infrastructure Layer** → Repositories & EF Core

🎯 الهدف: تطبيق Separation of Concerns.

---

## 3️⃣ Add The Base (Generic) Repository

Generic Repository بيحتوي على العمليات العامة (CRUD).

```csharp
public interface IRepository<T> where T : class
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Update(T entity);
    void Delete(T entity);
}
```

🎯 الهدف: إعادة استخدام الكود مع كل الـ Entities.

---

## 4️⃣ Test The Base (Generic) Repository

بنختبر الـ Generic Repository داخل Controller أو Service.

* Add
* GetAll
* GetById

```csharp
[HttpGet]
public IActionResult GetAllBooks()
{
    var books = unitOfWork.BookRepository.GetAll();
    return Ok(books);
}



[HttpGet("{id}")]
public IActionResult GetbookById(int id)
{
    var book = unitOfWork.BookRepository.GetById(id);
    if (book == null)
    {
        return NotFound();
    }
    return Ok(book);
}

```


🎯 الهدف: التأكد إن Repository شغالة بشكل سليم.

---

## 5️⃣ Add More Options – Part 1

إضافة Options إضافية مثل:

* Find باستخدام Condition
* FirstOrDefault

```csharp
IEnumerable<T> Find(Expression<Func<T, bool>> criteria);
```

🎯 الهدف: مرونة أكبر في الاستعلامات.

---

## 6️⃣ Add More Options – Part 2

إضافة دعم:

* Include (Eager Loading)
* OrderBy
```csharp
  IEnumerable<T> Find(Expression<Func<T, bool>>? match,List<string> includes,
      Expression<Func<T, object>> OrderBy, string OrderByType = OrderBy.Asc);
```


🎯 الهدف: التحكم في البيانات المرتجعة من Repository.

---

## 7️⃣ Add More Options – Part 3

استكمال الخيارات:

* Pagination
* Tracking / No Tracking
  
* ```csharp

  IEnumerable<T> Find(Expression<Func<T, bool>>? match,
       int PageNumber,
       List<string> includes,
      Expression<Func<T, object>> OrderBy
      , string OrderByType = OrderBy.Asc, int PageSize=3,bool withTracking=false);
  ```


🎯 الهدف: تحسين الأداء وتنظيم النتائج.

---

## 8️⃣ Add Unit Of Work

**Unit Of Work** مسؤول عن:

* تجميع كل Repositories
* تنفيذ SaveChanges مرة واحدة

```csharp
public interface IUnitOfWork : IDisposable
{
    IBookRepository Books { get; }
    IAuthorRepository Authors { get; }
    int Complete();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext context;

    public UnitOfWork(ApplicationDbContext context)
    {
        this.context = context;

        Books=new BookRepository(context);
        Authors=new AuthorRepository(context);
    }
    IBookRepository Books { get; }
    IAuthorRepository Authors { get; }

    public int Complete()
    {
        return context.SaveChanges();
    }

    public void Dispose()
    {
         context.Dispose();
    }
}
```

🎯 الهدف: التحكم في Transaction واحدة.

---

## 9️⃣ Testing The Unit Of Work

بنختبر:

* Add بيانات من أكتر من Repository
* تنفيذ `Complete()` مرة واحدة

🎯 الهدف: التأكد إن كل العمليات تتم مع بعض بنجاح.

---

## 🔟 Add Special Methods

إضافة Methods خاصة بكل Entity.

مثال:

```csharp
Movie GetbookWithAuthor(int id);
```

🎯 الهدف: تخصيص Repository حسب الحاجة.

---

## 1️⃣1️⃣ Use Complete Method

`Complete()` هي المسؤولة عن حفظ كل التغييرات.

```csharp
_unitOfWork.Complete();
```

⚠️ بدونها، أي Add / Update / Delete مش هيتحفظ.

🎯 الهدف: تنفيذ كل العمليات في Transaction واحدة.

---

