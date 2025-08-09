<pre>
<div align="right">
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
        

    </pre>
    </div>
