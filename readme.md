## What is TakNotify

`TakNotify` is an open source library for .NET Core applications to simplify sending notifications through various providers.

The main goal of `TakNotify` is to help developers to integrate various notification services into an application easily in a uniformed style, both for the setup mechanism and also for the send method invocation.

## Installation

`TakNotify` libraries are available as NuGet packages and can be installed easily into an ASP.NET Core application by using the `dotnet add package` command:

```powershell
dotnet add package TakNotify.AspNetCore
dotnet add package TakNotify.Provider.Smtp
```

The `TakNotify.AspNetCore` package contains the functionality to register the `INotification` object into the app's Dependency Injection system. `INotification` is the object in `TakNotify` that responsibles to manage the instance of TakNotify Providers and invoke the correct send notification method for you.

The `TakNotify.Provider.Smtp` package is an example of TakNotify Provider which will be used to send email notifications via SMTP. You could install multiple providers into an application.

Both packages above have dependencies to the main `TakNotify` library. But you don't have to install it separately because it will be downloaded automatically if you use the NuGet package installation command as above.

## Setup

After installing the required packages, you can add the following code to the `ConfigureServices` method in `Startup.cs`:

```c#
public void ConfigureServices(IServiceCollection services)
{
    ...

    services
        .AddGoNotify()
        .AddProvider<SmtpProvider, SmtpProviderOptions>(options =>
        {
            options.Server = Configuration.GetValue<string>("Smtp:Server");
            options.Port = Configuration.GetValue<int>("Smtp:Port");
            options.Username = Configuration.GetValue<string>("Smtp:Username");
            options.Password = Configuration.GetValue<string>("Smtp:Password");
            options.UseSSL = Configuration.GetValue<bool>("Smtp:UseSSL");
        });

    ...
}
```

The `AddGoNotify()` method will register the `INotification` as a singleton object in the application.

The `AddProvider<>()` method will register the notification provider along with its specific options. In the sample code above, the option values are taken from the `Configuration` object. You can add multiple providers in the set up.

## Sending the notification

Let's say you want to send notification from a controller, you can inject the `INotification` object into the constructor:

```c#
private readonly INotification _notification;
public WeatherForecastController(INotification notification)
{
    _notification = notification;
}
```

You can always use the generic `INotification.Send()` method to send the notification.
However, the notification providers that you have installed usually bring an extension method that you can use to send a notification that is specific to the provider. Like in the SMTP provider, you can use the `SendEmailWithSmtp()` method to send an email notification:

```c#
public async Task<IAsyncResult> Get()
{
    ...

    var message = new EmailMessage()
    {
        ToAddresses = new List<string> { "user@example.com" },
        FromAddress = "noreply@example.com",
        Subject = "[TakNotify] Weather Forecast",
        Body = $"Forecast: {JsonConvert.SerializeObject(items)}"
    };
    var result = await _notification.SendEmailWithSmtp(message);

    ...
}
```

## Available Providers

These are the providers that you can use with `TakNotify`:

- [TakNotify.Provider.SendGrid](https://www.nuget.org/packages/TakNotify.Provider.SendGrid/)
- [TakNotify.Provider.Smtp](https://www.nuget.org/packages/TakNotify.Provider.Smtp/)
- [TakNotify.Provider.Twilio](https://www.nuget.org/packages/TakNotify.Provider.Twilio/)

More providers will be available soon.

## Build from the source code

The source code of `TakNotify` can be cloned from [GitHub](https://github.com/TakNotify/TakNotify):

```powershell
git clone https://github.com/TakNotify/TakNotify
```

The easiest way to build `TakNotify` from the source code is by executing the `build.ps1` script:

```powershell
.\build.ps1
```

The only pre-requisite to build `TakNotify` is that you have `.NET Core SDK 3.1` installed in your machine.
