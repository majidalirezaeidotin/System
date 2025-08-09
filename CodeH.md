<pre>
<div dir="rtl" align="right">
services.AddSingleton<IMyService, MyService>();  // برای کانفیگ یا کش مناسب است
services.AddScoped<IMyService, MyService>();     // برای سرویس‌های وابسته به کاربر
services.AddTransient<IMyService, MyService>();  // برای استفاده‌های کوتاه‌مدت
-------------------------
Middlewareها در ASP.NET Core اجزایی هستند که درخواست‌های HTTP را در pipeline دریافت، پردازش و به مرحله بعدی ارسال می‌کنند. هر middleware می‌تواند:
درخواست را پردازش کند.
تصمیم بگیرد که آیا درخواست به middleware بعدی ارسال شود یا نه.
پاسخ را اصلاح کند.
----------------------------
Mediator یک الگو (Pattern) برای کاهش وابستگی مستقیم بین اجزا است. در ASP.NET Core، کتابخانه‌ای مانند MediatR برای پیاده‌سازی آن استفاده می‌شود.
مزایا:
جداسازی لایه‌ها
افزایش تست‌پذیری
کاهش پیچیدگی در ارتباط بین Handlerها و Controllerها
مثال:
public record GetUserQuery(int Id) : IRequest<User>;
public class GetUserHandler : IRequestHandler<GetUserQuery, User>
{
    public Task<User> Handle(GetUserQuery request, CancellationToken cancellationToken)
    {
        return GetUserFromDb(request.Id);
    }
}
-------------------------
سرویس‌های scoped برای هر درخواست ساخته می‌شوند، اما BackgroundService خارج از request context اجرا می‌شود، و به همین دلیل نمی‌تواند مستقیماً scoped service را inject کند.

🔧 راه‌حل:
ایجاد یک scope دستی:


using (var scope = _serviceProvider.CreateScope())
{
    var myService = scope.ServiceProvider.GetRequiredService<IMyScopedService>();
    // استفاده از سرویس
}
-----------------------------------------------
چگونه می‌توان دسترسی به route خاصی را فقط به کاربران خاصی داد؟
پاسخ:
استفاده از Attribute-based authorization:


[Authorize(Roles = "Admin")]
public IActionResult AdminOnly()
{
    return View();
}
یا policy-based authorization:

csharp
Copy
Edit
services.AddAuthorization(options =>
{
    options.AddPolicy("IsManager", policy =>
        policy.RequireClaim("Position", "Manager"));
});

[Authorize(Policy = "IsManager")]
---------------------------------------------

تفاوت بین IHostedService و BackgroundService در ASP.NET Core چیست؟
پاسخ:

IHostedService: یک اینترفیس ساده است که متدهای StartAsync و StopAsync دارد و باید به‌صورت کامل پیاده‌سازی شود.

BackgroundService: یک کلاس انتزاعی است که IHostedService را پیاده‌سازی کرده و متد ExecuteAsync را برای اجرای حلقه اصلی کارها فراهم می‌کند.
📌 نکته: اگر کار طولانی‌مدت و تکراری داریم، BackgroundService ساده‌تر و خواناتر است.        


-----------------------------------        
چه زمانی استفاده از ValueTask به جای Task مفید است؟
پاسخ:
ValueTask برای زمانی مناسب است که:

گاهی نتیجه بلافاصله آماده است (بدون async).

می‌خواهیم از ساخت شیء اضافی Task جلوگیری کنیم.
📌 هشدار: استفاده نابجا باعث پیچیدگی و افت کارایی می‌شود، مخصوصاً وقتی چند بار await شود.        
---------------------------------------------

تفاوت IOptions<T>، IOptionsSnapshot<T> و IOptionsMonitor<T> چیست؟
پاسخ:

IOptions<T>: فقط یک‌بار در طول عمر برنامه مقداردهی می‌شود (Singleton).

IOptionsSnapshot<T>: برای هر Request مقدار تازه از کانفیگ می‌گیرد (Scoped).

IOptionsMonitor<T>: هر زمان فایل کانفیگ تغییر کند، مقدار جدید را بدون نیاز به ری‌استارت برمی‌گرداند.
--------------------------------------------------

در ASP.NET Core چگونه می‌توان جریان درخواست را به‌طور موقت متوقف و دوباره ادامه داد؟
پاسخ:
با استفاده از Middleware و متدهای async:
await Task.Delay(5000); // توقف موقت
await next(); // ادامه پردازش
📌 کاربرد: سناریوهای Throttling یا شبیه‌سازی تأخیر شبکه.    
---------------------------------------
تفاوت بین Func<>، Action<> و Predicate<> در C# چیست؟
پاسخ:

Func<in T, out TResult>: متدی با مقدار بازگشتی دارد.

Action<in T>: متدی بدون مقدار بازگشتی است.

Predicate<T>: متدی که bool برمی‌گرداند (برای شرط‌گذاری).
--------------------------------------------------
در HttpClientFactory چگونه می‌توان یک Handler سفارشی برای همه درخواست‌ها اضافه کرد؟
services.AddHttpClient("MyClient")
    .AddHttpMessageHandler<MyCustomHandler>();
📌 این روش برای افزودن قابلیت‌هایی مثل Retry، Logging یا Authentication مناسب است.
------------------------------------------------
تفاوت بین In-Process و Out-of-Process hosting در ASP.NET Core چیست؟
In-Process: اپلیکیشن مستقیماً در فرآیند IIS اجرا می‌شود. (سریع‌تر)

Out-of-Process: اپلیکیشن در Kestrel اجرا می‌شود و IIS به‌عنوان Reverse Proxy عمل می‌کند. (انعطاف‌پذیرتر)
📌 پیش‌فرض از نسخه 2.2 به بعد، In-Process است.        
--------------------------------------------------
چه تفاوتی بین CancellationTokenSource.Cancel() و CancellationTokenSource.Dispose() وجود دارد؟
Cancel(): سیگنال لغو عملیات Async را ارسال می‌کند.

Dispose(): منابع داخلی Token را آزاد می‌کند.
📌 نکته: بهتر است بعد از Cancel همیشه Dispose هم انجام شود.        
----------------------------------------------
در ASP.NET Core چه زمانی استفاده از MapWhen در Routing توصیه می‌شود؟
وقتی می‌خواهیم مسیرهای خاصی را بر اساس یک شرط (مثلاً هدر یا کوئری‌استرینگ) پردازش کنیم.
app.MapWhen(ctx => ctx.Request.Query.ContainsKey("debug"), appBuilder =>
{
    appBuilder.Run(async context =>
    {
        await context.Response.WriteAsync("Debug Mode");
    });
});        
----------------------------------------------
ه زمانی استفاده از record به جای class مناسب‌تر است؟
وقتی هدف اصلی نگهداری داده‌ها و کار با اشیاء Immutable است.
پیاده‌سازی خودکار Equals و GetHashCode

پشتیبانی از with-expression برای ساخت نسخه‌های تغییر یافته
public record User(string Name, int Age);        
-------------------------------------------------

        
    </pre>
    </div>
