### Creating from template (automatic way)

The easiest way of starting a new project using ABP and module-zero, to
create a template on [templates page](/Templates). See [startup
template](/Pages/Documents/Zero/Startup-Template) documentation.

### Installing manually

If you have a pre-created application and install module-zero later, you
can follow the instructions in this section.

In this document, I will assume that your solution has these projects:

-   AbpZeroSample.Core
-   AbpZeroSample.Application
-   AbpZeroSample.EntityFramework
-   AbpZeroSample.Web
-   AbpZeroSample.WebApi

#### Core (domain) layer

Install **Abp.Zero** nuget package to .Core project. Then go to the core
module class (AbpZeroSampleCoreModule class for this sample) and add
**DependsOn** attribute for **AbpZeroCoreModule** as shown below:

    [DependsOn(typeof(AbpZeroCoreModule))]
    public class AbpZeroSampleCoreModule : AbpModule
    {
        public override void Initialize()
        {
            IocManager.RegisterAssemblyByConvention(Assembly.GetExecutingAssembly());
        }
    }

##### Domain classes (entities)

Module-zero provides **User**, **Role** and **Tenant** classes as
abstract. So, we should implement they as shown below:

    public class User : AbpUser<Tenant, User>
    {

    }

    public class Role : AbpRole<Tenant, User>
    {

    }

    public class Tenant : AbpTenant<Tenant, User>
    {

    }

You can add your custom properties here. In this way, we can extend base
user, role and tenant classes upon our needs.

##### Managers (domain services)

We should implement base **manager** and **store** classes since they
are also abstract.

Start from **user store** and **user manager**:

    public class UserStore : AbpUserStore<Tenant, Role, User>
    {
        public UserStore(
            IRepository<User, long> userRepository,
            IRepository<UserLogin, long> userLoginRepository,
            IRepository<UserRole, long> userRoleRepository,
            IRepository<Role> roleRepository,
            IRepository<UserPermissionSetting, long> userPermissionSettingRepository,
            IUnitOfWorkManager unitOfWorkManager
            )
            : base(
                userRepository,
                userLoginRepository,
                userRoleRepository,
                roleRepository,
                userPermissionSettingRepository,
                unitOfWorkManager
            )
        {
        }
    }

    public class UserManager : AbpUserManager<Tenant, Role, User>
    {
        public UserManager(
            UserStore userStore,
            RoleManager roleManager,
            IRepository<Tenant> tenantRepository,
            IMultiTenancyConfig multiTenancyConfig,
            IPermissionManager permissionManager,
            IUnitOfWorkManager unitOfWorkManager,
            ISettingManager settingManager,
            IUserManagementConfig userManagementConfig,
            IIocResolver iocResolver,
            ICacheManager cacheManager,
            IRepository<OrganizationUnit, long> organizationUnitRepository,
            IRepository<UserOrganizationUnit, long> userOrganizationUnitRepository,
            IOrganizationUnitSettings organizationUnitSettings)
            : base(
                userStore,
                roleManager,
                tenantRepository,
                multiTenancyConfig,
                permissionManager,
                unitOfWorkManager,
                settingManager,
                userManagementConfig,
                iocResolver,
                cacheManager,
                organizationUnitRepository,
                userOrganizationUnitRepository,
                organizationUnitSettings)
        {

        }
    }

Don't worry dependency list. They may change in next versions. Just
arrange constructors if needed (or you can copy from [this
project](https://github.com/aspnetboilerplate/module-zero/tree/dev/test/Abp.Zero.SampleApp)).
It's similar for **role store** and **role manager**:

    public class RoleStore : AbpRoleStore<Tenant, Role, User>
    {
        public RoleStore(
            IRepository<Role> roleRepository,
            IRepository<UserRole, long> userRoleRepository,
            IRepository<RolePermissionSetting, long> rolePermissionSettingRepository)
            : base(
                roleRepository,
                userRoleRepository,
                rolePermissionSettingRepository
            )
        {

        }
    }

    public class RoleManager : AbpRoleManager<Tenant, Role, User>
    {
        public RoleManager(
            RoleStore store, 
            IPermissionManager permissionManager, 
            IRoleManagementConfig roleManagementConfig, 
            ICacheManager cacheManager)
            : base(
                store, 
                permissionManager, 
                roleManagementConfig, 
                cacheManager)
        {
        }
    }

Here, the **tenant manager** (no tenant store here):

    public class TenantManager : AbpTenantManager<Tenant, Role, User>
    {
        public TenantManager(
            IRepository<Tenant> tenantRepository, 
            IRepository<TenantFeatureSetting, long> tenantFeatureRepository, 
            EditionManager editionManager) : 
            base(
                tenantRepository, 
                tenantFeatureRepository, 
                editionManager
            )
        {
        }
    }

And finally, **feature value store** and **edition manager**:

    public class FeatureValueStore : AbpFeatureValueStore<Tenant, Role, User>
    {
        public FeatureValueStore(TenantManager tenantManager)
            : base(tenantManager)
        {

        }
    }

    public class EditionManager : AbpEditionManager
    {
        public const string DefaultEditionName = "Standard";

        public EditionManager(
            IRepository<Edition> editionRepository, 
            IRepository<EditionFeatureSetting, long> editionFeatureRepository) 
            : base(
                editionRepository, 
                editionFeatureRepository
            )
        {

        }
    }

##### Permission checker

To make [authorization](/Pages/Documents/Authorization) system work, we
should implement **permission checker**:

    public class PermissionChecker : PermissionChecker<Tenant, Role, User>
    {
        public PermissionChecker(UserManager userManager)
            : base(userManager)
        {

        }
    }

#### Infrastructure layer

##### EntityFramework

If you selected EntityFramework, we should configure it to use
module-zero. Install **Abp.Zero.EntityFramework** nuget package to
.EntityFramework project. Then go to module file
(AbpZeroSampleDataModule for this sample) and change
AbpEntityFrameworkModule dependency to **AbpZeroEntityFrameworkModule**
as shown below:

    [DependsOn(typeof(AbpZeroEntityFrameworkModule), typeof(AbpZeroSampleCoreModule))]
    public class AbpZeroSampleDataModule : AbpModule
    {
        //...
    }

###### DbContext

Go to your DbContext class and change base class from AbpDbContext to
**AbpZeroDbContext** as shown below:

    public class AbpZeroSampleDbContext : AbpZeroDbContext<Tenant, Role, User>
    {
        //...
    }

Thus, base entities from module-zero is added to your db context.

###### Database Migration

Now, we should create database migrations since our db context is
changed. **Open Package Manager Console** and type the following
command:

    Add-Migration "AbpZero_Installed"

Surely, you can select a different migration name. Don't forget to
select Default Project to AbpZeroSample.EntityFramework in package
manager console (AbpZeroSample will be different for your case). After
executing this command, a new migration file is added to the project.
Check it and change if you need. Then type the following command to
update database schema:

    Update-Database

You can check EntityFramework's code-first migration
[documentation](https://msdn.microsoft.com/en-us/data/jj591621.aspx) for
more information.

###### Initial Data

If you check your database, you will see that tables are created but
they are empty. You can use EntityFramework's **seed** to fill initial
data. You can use such a class as initial data builder:

    public class DefaultTenantRoleAndUserBuilder
    {
        private readonly AbpZeroSampleDbContext _context;

        public DefaultTenantRoleAndUserBuilder(AbpZeroSampleDbContext context)
        {
            _context = context;
        }

        public void Build()
        {
            CreateUserAndRoles();
        }

        private void CreateUserAndRoles()
        {
            //Admin role for tenancy owner

            var adminRoleForTenancyOwner = _context.Roles.FirstOrDefault(r => r.TenantId == null && r.Name == "Admin");
            if (adminRoleForTenancyOwner == null)
            {
                adminRoleForTenancyOwner = _context.Roles.Add(new Role {Name = "Admin", DisplayName = "Admin"});
                _context.SaveChanges();
            }

            //Admin user for tenancy owner

            var adminUserForTenancyOwner = _context.Users.FirstOrDefault(u => u.TenantId == null && u.UserName == "admin");
            if (adminUserForTenancyOwner == null)
            {
                adminUserForTenancyOwner = _context.Users.Add(
                    new User
                    {
                        TenantId = null,
                        UserName = "admin",
                        Name = "System",
                        Surname = "Administrator",
                        EmailAddress = "admin@aspnetboilerplate.com",
                        IsEmailConfirmed = true,
                        Password = "AM4OLBpptxBYmM79lGOX9egzZk3vIQU3d/gFCJzaBjAPXzYIK3tQ2N7X4fcrHtElTw==" //123qwe
                    });

                _context.SaveChanges();

                _context.UserRoles.Add(new UserRole(adminUserForTenancyOwner.Id, adminRoleForTenancyOwner.Id));

                _context.SaveChanges();
            }

            //Default tenant

            var defaultTenant = _context.Tenants.FirstOrDefault(t => t.TenancyName == "Default");
            if (defaultTenant == null)
            {
                defaultTenant = _context.Tenants.Add(new Tenant {TenancyName = "Default", Name = "Default"});
                _context.SaveChanges();
            }

            //Admin role for 'Default' tenant

            var adminRoleForDefaultTenant = _context.Roles.FirstOrDefault(r => r.TenantId == defaultTenant.Id && r.Name == "Admin");
            if (adminRoleForDefaultTenant == null)
            {
                adminRoleForDefaultTenant = _context.Roles.Add(new Role { TenantId = defaultTenant.Id, Name = "Admin", DisplayName = "Admin" });
                _context.SaveChanges();
            }

            //Admin for 'Default' tenant

            var adminUserForDefaultTenant = _context.Users.FirstOrDefault(u => u.TenantId == defaultTenant.Id && u.UserName == "admin");
            if (adminUserForDefaultTenant == null)
            {
                adminUserForDefaultTenant = _context.Users.Add(
                    new User
                    {
                        TenantId = defaultTenant.Id,
                        UserName = "admin",
                        Name = "System",
                        Surname = "Administrator",
                        EmailAddress = "admin@aspnetboilerplate.com",
                        IsEmailConfirmed = true,
                        Password = "AM4OLBpptxBYmM79lGOX9egzZk3vIQU3d/gFCJzaBjAPXzYIK3tQ2N7X4fcrHtElTw==" //123qwe
                    });
                _context.SaveChanges();

                _context.UserRoles.Add(new UserRole(adminUserForDefaultTenant.Id, adminRoleForDefaultTenant.Id));
                _context.SaveChanges();
            }
        }
    }

This class creates default tenant, roles and users. We can use it in
EF's **Configuration** class to seed our database:

    internal sealed class Configuration : DbMigrationsConfiguration<AbpZeroSample.EntityFramework.AbpZeroSampleDbContext>
    {
        public Configuration()
        {
            AutomaticMigrationsEnabled = false;
            ContextKey = "AbpZeroSample";
        }

        protected override void Seed(AbpZeroSample.EntityFramework.AbpZeroSampleDbContext context)
        {
            context.DisableAllFilters();
            new DefaultTenantRoleAndUserBuilder(context).Build();
        }
                    }

Here, we disabled data filters (so we can freely manipulate database)
and used the initial data builder class.

#### Presentation Layer

##### Nuget Packages

Add the following nuget packages to the .Web project:

-   Abp.Zero.EntityFramework (this will also add Abp.Zero and all
    dependencies)
-   Microsoft.AspNet.Identity.Owin
-   Microsoft.Owin.Host.SystemWeb

##### Owin Startup Class

Add an Owin Startup class like that:

    using AbpZeroSample.Web;
    using Microsoft.AspNet.Identity;
    using Microsoft.Owin;
    using Microsoft.Owin.Security.Cookies;
    using Owin;

    [assembly: OwinStartup(typeof(Startup))]
    namespace AbpZeroSample.Web
    {
        public class Startup
        {
            public void Configuration(IAppBuilder app)
            {
                // Enable the application to use a cookie to store information for the signed in user
                app.UseCookieAuthentication(new CookieAuthenticationOptions
                {
                    AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
                    LoginPath = new PathString("/Account/Login")
                });

                // Use a cookie to temporarily store information about a user logging in with a third party login provider
                app.UseExternalSignInCookie(DefaultAuthenticationTypes.ExternalCookie);
            }
        }
    }

##### Account Controller

We can create a Controller for login/logout as shown below:

    public class AccountController : AbpZeroSampleControllerBase
    {
        private readonly UserManager _userManager;

        private IAuthenticationManager AuthenticationManager
        {
            get
            {
                return HttpContext.GetOwinContext().Authentication;
            }
        }

        public AccountController(UserManager userManager)
        {
            _userManager = userManager;
        }

        public ActionResult Login(string returnUrl = "")
        {
            if (string.IsNullOrWhiteSpace(returnUrl))
            {
                returnUrl = Request.ApplicationPath;
            }

            ViewBag.ReturnUrl = returnUrl;

            return View();
        }

        [HttpPost]
        public async Task<JsonResult> Login(LoginViewModel loginModel, string returnUrl = "")
        {
            if (!ModelState.IsValid)
            {
                throw new UserFriendlyException("Your form is invalid!");
            }

            var loginResult = await _userManager.LoginAsync(
                loginModel.UsernameOrEmailAddress,
                loginModel.Password,
                loginModel.TenancyName
                );

            switch (loginResult.Result)
            {
                case AbpLoginResultType.Success:
                    break;
                case AbpLoginResultType.InvalidUserNameOrEmailAddress:
                case AbpLoginResultType.InvalidPassword:
                    throw new UserFriendlyException("Invalid user name or password!");
                case AbpLoginResultType.InvalidTenancyName:
                    throw new UserFriendlyException("No tenant with name: " + loginModel.TenancyName);
                case AbpLoginResultType.TenantIsNotActive:
                    throw new UserFriendlyException("Tenant is not active: " + loginModel.TenancyName);
                case AbpLoginResultType.UserIsNotActive:
                    throw new UserFriendlyException("User is not active: " + loginModel.UsernameOrEmailAddress);
                case AbpLoginResultType.UserEmailIsNotConfirmed:
                    throw new UserFriendlyException("Your email address is not confirmed!");
                default: //Can not fall to default for now. But other result types can be added in the future and we may forget to handle it
                    throw new UserFriendlyException("Unknown problem with login: " + loginResult.Result);
            }

            AuthenticationManager.SignOut(DefaultAuthenticationTypes.ExternalCookie);
            AuthenticationManager.SignIn(new AuthenticationProperties { IsPersistent = loginModel.RememberMe }, loginResult.Identity);

            if (string.IsNullOrWhiteSpace(returnUrl))
            {
                returnUrl = Request.ApplicationPath;
            }

            return Json(new MvcAjaxResponse { TargetUrl = returnUrl });
        }

        public ActionResult Logout()
        {
            AuthenticationManager.SignOut();
            return RedirectToAction("Login");
        }
    }

With a simple LoginViewModel:

    public class LoginViewModel
    {
        public string TenancyName { get; set; }

        [Required]
        public string UsernameOrEmailAddress { get; set; }

        [Required]
        public string Password { get; set; }

        public bool RememberMe { get; set; }
    }

##### Login View

To be able to use AccountController, we should create a login page. It's
up to you creating a login form. Just call AccountController.Login via
AJAX with appropriate parameters.

##### Secure The Application

Now, we can add an AbpAuthorize attribute to HomeController to ensure
only authenticated users can enter to the page:

    [AbpMvcAuthorize]
    public class HomeController : AbpZeroSampleControllerBase
    {
        public ActionResult Index()
        { 
            return View("~/App/Main/views/layout/layout.cshtml"); //Layout of the angular application.
        }
    }
