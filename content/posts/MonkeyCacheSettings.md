+++
date = '2025-09-10T10:45:20-04:00'
draft = true
title = 'Monkey Cache Settings'
+++

## What is Monkey Cache?

**MonkeyCache** is a lightweight caching library for .NET applications, created. It’s designed to make caching flexible and straightforward—especially useful when you want to store data temporarily without setting up a full-blown database or complex infrastructure.



### Key Features

- **Simple API**: Uses a concept called a "Barrel" to store and retrieve cached items.

- **Time-based expiration**: Cached items can specify how long an item should stay in the cache.

- **Multiple storage backends:** 
  
  - `MonkeyCahce.FileStore`: Store data in local files.
  
  - `MonkeyCache.SQLite`: Uses SQLite for structured caching.
  
  - `MonkeyCache.LiteDB`: Uses LiteDB for document-style caching.

- **Cross-platform support**: Can be used with all applications built on .NET 8+.

### Why use it?

Perfect for scenarios like:

- Caching API responses

- Storing user preferences or session data

- Reducing redundant network calls in mobile apps.

It’s convenient in mobile or desktop apps where you want quick, local caching without spinning up a whole database.



## How do I use it:

#### Initialization:

I usually have a multi-project solution and multiple projects running when I launch the application. Hence, to limit my code repetition, I initialize MonkeyCache using the following code:

```csharp
public static class CacheInitialize
{
    public static void Initialize(string applicationName)
    {
        Barrel.ApplicationId = applicationName;
        var cachePath = Environment.GetEnvironmentVariable("HOME");
        if (RuntimeInformation.IsOSPlatform(OSPlatform.Linux))
        {
            cachePath += $"/cache/DbData/{applicationName}";
            DirectoryInfo di;
            if (!Directory.Exists(cachePath))
            {
                di = Directory.CreateDirectory(cachePath);
            }
            else
            {
                di = new DirectoryInfo(cachePath);
            }
            if (di.Exists)
            {
                Barrel.Create(cachePath);
            }
        }
        else
        {
            cachePath = Path.GetTempPath();
            cachePath = Path.Combine(cachePath, applicationName);
            Barrel.Create(cachePath);
        }
    }
}
```

The code is designed to operate seamlessly on both Linux and Windows platforms. It detects the operating system and creates the database file accordingly. Please note that the application establishes the cache within a directory named after the application.    

#### Storing and data retrieval:

The following code is the one I use for data storage and retrieval. 

```csharp
public static class HandleCache
{
    private static Lock? lockingObj;
    private static IBarrel? currentBarrel;

    public static string? GetString(string key)
    {
        currentBarrel ??= Barrel.Current;
        lockingObj ??= new Lock();
        lock (lockingObj)
        {
            if (!currentBarrel.Exists(key) || currentBarrel.IsExpired(key))
            {
                currentBarrel.EmptyExpired();
                return null;
            }
            return currentBarrel.Get<string>(key);
        }
    }

    public static void SaveString(string key, string value, TimeSpan timeSpan)
    {
        currentBarrel ??= Barrel.Current;
        lockingObj ??= new Lock();
        lock (lockingObj)
        {
            currentBarrel.Add(key, value, timeSpan);
        }
    }
}
```

- I'm using a simple object locking to ensure that it can be used in a multi-process application. I had issues in the past with multi-process applications and still maintain the lock.

- This code saves and retrieves only a string object. I use `JsonSerializer.Deserialize` and `JsonSerializer.Serialize` to convert my objects into string objects. 

- In some of the applications, I use Templates, but the code is a little complex. Sharing the Template code below:

```csharp
public class HandleCache : IHandleCache
{
    private static object? lockingObj;
    private readonly IBarrel currentBarrel;
    private readonly HttpClient httpClient;
    private readonly ILogger<HandleCache> logger;

    public HandleCache(HttpClient httpClient,
                       ILogger<HandleCache> logger)
    {
        this.httpClient = httpClient;
        httpClient.Timeout = TimeSpan.FromSeconds(5);
        this.logger = logger;
        currentBarrel = Barrel.Current;
    }

    public async Task<T> GetAsync<T>(string url, CacheDuration cacheDuration = CacheDuration.Minutes, int duration = 60,
                                     bool forceRefresh = false, string nullReplace = "")
    {
        string json = await GetStringAsync(url, cacheDuration, duration, forceRefresh);
        string pattern = @"Date\"": \""None\""";
        string substitution = "Date\": \"1900-01-01\"";
        RegexOptions options = RegexOptions.Multiline;
        Regex regex = new Regex(pattern, options);
        json = regex.Replace(json, substitution);
        json = JsonNode.Parse(json)!.ToString();
        if (!string.IsNullOrWhiteSpace(nullReplace))
        {
            json = Regex.Replace(json, nullReplace, "\"0\"");
        }
        try
        {
            if (!string.IsNullOrWhiteSpace(json))
            {
                T? retObj = JsonSerializer.Deserialize<T>(json);
                if (retObj != null)
                {
                    return retObj;
                }
            }
        }
        catch (Exception ex)
        {
            logger.LogError($"Error while processing data from {url}\n");
            logger.LogError(ex.Message);
        }
        return (T)Activator.CreateInstance(typeof(T), Array.Empty<object>())!;
    }

    public async Task<string> GetStringAsync(string url, CacheDuration cacheDuration = CacheDuration.Minutes,
                                             int duration = 60, bool forceRefresh = false)
    {
        string json = string.Empty;
        try
        {
            lockingObj ??= new object();
            lock (lockingObj)
            {
                currentBarrel.EmptyExpired();
            }
            if (!forceRefresh && !currentBarrel.IsExpired(url))
            {
                json = currentBarrel.Get<string>(url);
            }
            if (string.IsNullOrWhiteSpace(json))
            {
                json = await httpClient.GetStringAsync(url);
                lock (lockingObj)
                {
                    var expiring = cacheDuration switch
                    {
                        CacheDuration.Seconds => TimeSpan.FromSeconds(duration),
                        CacheDuration.Minutes => TimeSpan.FromSeconds(duration),
                        CacheDuration.Hours => TimeSpan.FromHours(duration),
                        CacheDuration.Days => TimeSpan.FromDays(duration),
                        _ => TimeSpan.FromMinutes(duration),
                    };
                    currentBarrel.Add(url, json, expiring);
                }
            }
            json = json.Replace(":\"N/A\"", ":null");
            return json;
        }
        catch (Exception ex)
        {
            logger.LogError($"Error while processing data from {url}", ex);
            return string.Empty;
        }
    }
}
```

And the interface I use to inject:

```csharp
public interface IHandleCache
{
    Task<T> GetAsync<T>(string url,
        CacheDuration cacheDuration = CacheDuration.Minutes,
        int duration = 60,
        bool forceRefresh = false,
        string nullReplace = "");

    Task<string> GetStringAsync(string url,
        CacheDuration cacheDuration = CacheDuration.Minutes,
        int duration = 60,
        bool forceRefresh = false);
}

public enum CacheDuration
{
    Seconds,
    Minutes,
    Hours,
    Days
}
```

Let's not go over the details of what this code does, but I can confirm it works. If you're interested, feel free to use it.
