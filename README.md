### Hitting a 3rd Party API from your server

Fetching data from your server instead of your client can be helpful for multiple reasons. For one, it can solve the issue of needing to access data from an API that does not enable CORS. Secondly, it allows us to keep sensitive data like api keys and account credentials out of your client side code where all users can see it.

To secure senitive information in your ASP.NET application, you can use the Secret Manager tool. To access it, right click on your project's name in visual studio solution explorer window and select "Manage User Secrets". This will open up a json file where you can store your sensitive material. For hitting the DarkSky API, I have mine set up to look something like this:

```javascript
{
  "ApiKeys": {
    "DarkSky": "my-actual-darksky-api-key-goes-here"
  }
}
```


To access these secrets in my Controller code I can do something like this:

```csharp
string key = _config["ApiKeys:DarkSky"];
```

Here is an example in my `HomeController`

```csharp
public class HomeController : Controller
{
    private readonly string _weatherUrl = "https://api.darksky.net/forecast/";
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }

    public async Task<IActionResult> Index()
    {
        var weather = await GetWeatherAsync();
        return Ok(weather);
    }

    private async Task<Weather> GetWeatherAsync()
    {
        var key = _config["ApiKeys:DarkSky"];
        var url = $"{_weatherUrl}{key}/36,-86";
        var client = new HttpClient();
        var response = await client.GetAsync(url);

        if (response.IsSuccessStatusCode)
        {
            var weather = await response.Content.ReadAsAsync<Weather>();
            return weather;
        }
        else
        {
            return null;
        }
    }
}
```


#### Secret Manager Documentation
https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-2.2&tabs=windows