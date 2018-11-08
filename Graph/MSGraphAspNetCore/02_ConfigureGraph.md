## Configuración Graph

1. Añade el paquete de NuGet __Microsoft.Identity.Client__ (el paquete es todavía una pre-release, así que asegurate de marcar la opción _Include prerelease_ en Nuget)
2. Añade el paquete de Nuget __Microsoft.Graph__ 


3. Abrimos el fichero **appsettings.json** y añadimos las siguientes claves en el nodo AzureAD

```
    "ClientSecret": "[introduce la contraseña que has generado al crear la aplicación]]", 
    "BaseUrl": "https://localhost:44334",
    "Scopes": "openid email profile offline_access",
    "GraphResourceId": "https://graph.microsoft.com/",
    "GraphScopes": "User.Read User.ReadBasic.All Mail.Send Files.ReadWrite"
```

> Aquí es donde vamos a configurar los permisos de Graph que requiere nuestra aplicación.

3. Abrimos el fichero **AzureAdOptions.cs** que se encuentra en la carpeta **Extensions** y añadimos las propiedades para manejar las claves que hemos introducido en el fichero appsettings.json

```
public string ClientSecret { get; set; }

public string BaseUrl { get; set; }

public string Scopes { get; set; }

public string GraphResourceId { get; set; }

public string GraphScopes { get; set; }
```

4. Creamos una carpeta que se llame **Helpers**
5. En la carpeta Helpers añadimos una nueva clase que se llame **SessionTokenCache**

   a. Sustitumos los using por

   ```
    using Microsoft.Extensions.Caching.Memory;
    using Microsoft.Identity.Client;
    using System.Text;
    ```

    b. y el contenido de la clase por

    ```
    public class SessionTokenCache
    {
        private static readonly object FileLock = new object();
        private readonly string _cacheId;
        private readonly IMemoryCache _memoryCache;
        private TokenCache _cache = new TokenCache();

        public SessionTokenCache(string userId, IMemoryCache memoryCache)
        {
            // not object, we want the SUB
            _cacheId = userId + "_TokenCache";
            _memoryCache = memoryCache;

            Load();
        }

        public TokenCache GetCacheInstance()
        {
            _cache.SetBeforeAccess(BeforeAccessNotification);
            _cache.SetAfterAccess(AfterAccessNotification);
            Load();

            return _cache;
        }

        public void SaveUserStateValue(string state)
        {
            lock (FileLock)
            {
                _memoryCache.Set(_cacheId + "_state", Encoding.ASCII.GetBytes(state));
            }
        }

        public string ReadUserStateValue()
        {
            string state;
            lock (FileLock)
            {
                state = Encoding.ASCII.GetString(_memoryCache.Get(_cacheId + "_state") as byte[]);
            }

            return state;
        }

        public void Load()
        {
            lock (FileLock)
            {
                _cache.Deserialize(_memoryCache.Get(_cacheId) as byte[]);
            }
        }

        public void Persist()
        {
            lock (FileLock)
            {
                // reflect changes in the persistent store
                _memoryCache.Set(_cacheId, _cache.Serialize());
                // once the write operation took place, restore the HasStateChanged bit to false
                _cache.HasStateChanged = false;
            }
        }

        // Empties the persistent store.
        public void Clear()
        {
            _cache = null;
            lock (FileLock)
            {
                _memoryCache.Remove(_cacheId);
            }
        }

        // Triggered right before MSAL needs to access the cache.
        // Reload the cache from the persistent store in case it changed since the last access.
        private void BeforeAccessNotification(TokenCacheNotificationArgs args)
        {
            Load();
        }

        // Triggered right after MSAL accessed the cache.
        private void AfterAccessNotification(TokenCacheNotificationArgs args)
        {
            // if the access operation resulted in a cache update
            if (_cache.HasStateChanged)
            {
                Persist();
            }
        }
    }
    ```
6. Dentro de la carpeta Helpers creamos una clase que se llame **GraphAuthProvider**
    a. Sustituimos la sección de using por
```
using Microsoft.AspNetCore.Authentication;
using Microsoft.Extensions.Caching.Memory;
using Microsoft.Extensions.Configuration;
using Microsoft.Graph;
using Microsoft.Identity.Client;
using System;
using System.Linq;
using System.Threading.Tasks;
```
b. y el contenido de la clase por

````
public class GraphAuthProvider : IGraphAuthProvider
    {
        private readonly IMemoryCache _memoryCache;
        private TokenCache _userTokenCache;

        // Properties used to get and manage an access token.
        private readonly string _appId;
        private readonly ClientCredential _credential;
        private readonly string[] _scopes;
        private readonly string _redirectUri;

        public GraphAuthProvider(IMemoryCache memoryCache, IConfiguration configuration)
        {
            var azureOptions = new AzureAdOptions();
            configuration.Bind("AzureAd", azureOptions);

            _appId = azureOptions.ClientId;
            _credential = new ClientCredential(azureOptions.ClientSecret);
            _scopes = azureOptions.GraphScopes.Split(new[] { ' ' });
            _redirectUri = azureOptions.BaseUrl + azureOptions.CallbackPath;

            _memoryCache = memoryCache;
        }

        // Gets an access token. First tries to get the access token from the token cache.
        // Using password (secret) to authenticate. Production apps should use a certificate.
        public async Task<string> GetUserAccessTokenAsync(string userId)
        {
            _userTokenCache = new SessionTokenCache(userId, _memoryCache).GetCacheInstance();

            var cca = new ConfidentialClientApplication(
                _appId,
                _redirectUri,
                _credential,
                _userTokenCache,
                null);

            var accounts = (await cca.GetAccountsAsync()).ToList();
            if (!accounts.Any()) throw new ServiceException(new Error
            {
                Code = "TokenNotFound",
                Message = "User not found in token cache. Maybe the server was restarted."
            });

            try
            {
                var result = await cca.AcquireTokenSilentAsync(_scopes, accounts.First());
                return result.AccessToken;
            }

            // Unable to retrieve the access token silently.
            catch (Exception)
            {
                throw new ServiceException(new Error
                {
                    Code = GraphErrorCode.AuthenticationFailure.ToString(),
                    Message = "Caller needs to authenticate. Unable to retrieve the access token silently."
                });
            }
        }
    }

    public interface IGraphAuthProvider
    {
        Task<string> GetUserAccessTokenAsync(string userId);
    }
````
7. Dentro de la carpeta Helpers vamos a crear una clase que se llame **GraphSdkHelper**.

    a. Dentro del fichero creamos una interfaz

````
public interface IGraphSdkHelper
{
    GraphServiceClient GetAuthenticatedClient(string userId);
}
````

b. Y una clase que implemente la interfaz que hemos creado

````
public class GraphSdkHelper : IGraphSdkHelper
    {
        private readonly IGraphAuthProvider _authProvider;
        private GraphServiceClient _graphClient;

        public GraphSdkHelper(IGraphAuthProvider authProvider)
        {
            _authProvider = authProvider;
        }

        // Get an authenticated Microsoft Graph Service client.
        public GraphServiceClient GetAuthenticatedClient(string userId)
        {
            _graphClient = new GraphServiceClient(new DelegateAuthenticationProvider(
                async requestMessage =>
                {
                    // Passing tenant ID to the sample auth provider to use as a cache key
                    var accessToken = await _authProvider.GetUserAccessTokenAsync(userId);

                    // Append the access token to the request
                    requestMessage.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

                }));

            return _graphClient;
        }
    }
````

8. Abrimos el fichero **Startup.cs**

8.1. En la clase Startup creamos las siguientes propiedades

````
public IConfiguration Configuration { get; }
public const string ObjectIdentifierType = "http://schemas.microsoft.com/identity/claims/objectidentifier";
public const string TenantIdType = "http://schemas.microsoft.com/identity/claims/tenantid";
````

8.2. Sutituye el método **ConfigureServices** por

````
public void ConfigureServices(IServiceCollection services)
        {
            //services.AddAuthentication(sharedOptions =>
            //{
            //    sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
            //    sharedOptions.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
            //})
            //.AddAzureAd(options => Configuration.Bind("AzureAd", options))
            //.AddCookie();

            services.Configure<CookiePolicyOptions>(options =>
            {
                // This lambda determines whether user consent for non-essential cookies is needed for a given request.
                options.CheckConsentNeeded = context => true;
                options.MinimumSameSitePolicy = SameSiteMode.None;
            });

            services.AddAuthentication(sharedOptions =>
            {
                sharedOptions.DefaultAuthenticateScheme = CookieAuthenticationDefaults.AuthenticationScheme;
                sharedOptions.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
                sharedOptions.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
            })
            .AddAzureAd(options => Configuration.Bind("AzureAd", options))
            .AddCookie();

            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

            // This sample uses an in-memory cache for tokens and subscriptions. Production apps will typically use some method of persistent storage.
            services.AddMemoryCache();
            services.AddSession();

            // Add application services.
            //services.AddSingleton<IConfiguration>(Configuration);
            services.AddSingleton<IGraphAuthProvider, GraphAuthProvider>();
            services.AddTransient<IGraphSdkHelper, GraphSdkHelper>();

            services.Configure<HstsOptions>(options =>
            {
                options.IncludeSubDomains = true;
                options.MaxAge = TimeSpan.FromDays(365);
            });
        }
````

8.3. Sustituye el método **Configure** por

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Error");
                app.UseHsts();
            }

            app.UseHttpsRedirection();
            app.UseStaticFiles();
            app.UseCookiePolicy();
            app.UseSession();
            app.UseAuthentication();

            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
        }
```
Ir a [3-Obtener los datos del usuario](./03_RetrieveUserProfile.md)
