---
title: TakNotify
---

## What is it?

`TakNotify` is an open source library for .NET Core apps to simplify sending notifications 
into various targets.

## Installation & Setup

The easiest way to integrate `TakNotify` into your ASP.NET Core app is by installing 
the NuGet packages:

```powershell
dotnet add package TakNotify.AspNetCore
dotnet add package TakNotify.Provider.Smtp
```

The `TakNotify.AspNetCore` package contains the functionality to register the 
`INotification` object into the app so you can utilize it to send the notifications
with the Dependency Injection mechanism.

The `TakNotify.Provider.Smtp` package is an example of TakNotify Provider which will 
be used to send email notifications via SMTP. You could install multiple providers 
into an application. You can find the list of available providers in the 
[providers page](./providers.md).

After installing the required packages, you can add the following code to the 
`ConfigureServices` method in `Startup.cs`:

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

The `AddGoNotify()` method will register the `INotification` service which can be 
be used to send the notifications.

The `AddProvider<>()` method will register the notification provider along with its 
specific options. In the sample code above, the option values are taken from 
the `Configuration` object. You can add multiple providers in the set up.

## Sending the notification

Let's say you want to send notification from a controller, you can inject the 
`INotification` object into the constructor:

```c#
private readonly INotification _notification;
public WeatherForecastController(INotification notification)
{
    _notification = notification;
}
```

You can always use the generic `INotification.Send()` method to send the notification.
However, the notification providers that you have installed usually bring an extension 
method that you can use to send a notification that is specific to the provider. Like 
in the SMTP provider, you can use `SendEmailWithSmtp()` method to send an email 
notification:

```c#
public async Task<IAsyncResult> Get() 
{
    ...

    var message = new EmailMessage()
    {
        ToAddresses = new List<string> { _configuration.GetValue<string>("SampleUsers") },
        FromAddress = _configuration.GetValue<string>("DefaultFromAddress"),
        Subject = "[TakNotify] Weather Forecast",
        Body = $"Forecast: {JsonConvert.SerializeObject(items)}"
    };
    var result = await _notification.SendEmailWithSmtp(message);

    ...
}
```


## Build from the source code

The source code of `TakNotify` can be cloned from https://github.com/TakNotify/TakNotify.

The easiest way to build `TakNotify` from the source code is by executing the `build.ps1` 
script:

```powershell
.\build.ps1
```

The only pre-requisite to build `TakNotify` is that you have .NET Core SDK 3.1 
installed in your machine.
