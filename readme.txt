### API
Quand AddDbContext<DataContext> est appelé, le runtime lui passe un DbContextOptionsBuilder en paramètre
On récupère ce DbContextOptionsBuilder et on utilise sa méthode UseSqlite pour créer les DbContextOptions avec un fonction lambda

services.AddDbContext<DataContext>(options =>
            {
                    options.UseSqlite(config.GetConnectionString("DefaultConnection"));
            });

AddDbContext : Action<DbContextOptionsBuilder>, action sur un DbContextOptionsBuilder


Quand app.UseCors est appelé, le runtime lui passe un CorsPolicyBuilder en paramètre
On récupère ce CorsPolicyBuilder et on invoque sa méthode AllowAnyHeader...


private async Task<bool> UserExists(string username)
{
    return await _context.Users.AnyAsync(x => x.UserName == username.ToLower());
}

Users = database
AnyAsyn = check for emptiness
Pour chaque éléments x de Users, on test si son nom == username

ActionResult : we can return different http code reseult


dotnet ef migrations add UserPasswordAdded
dotnet ef database update 
dotnet ef database drop


Routes:
    - Controllers/...


ServiceToken:
    - Services/TokenService.cs = create and return token (string), implémente l'interface ITokenService
    - Interfaces/ITokenService.cs = méthode CreateToken
    - Extensions/ApplicationServiceExtension.cs = retourne le service de Token
        - services.AddScoped<Interfaces.ITokenService, TokenService>();
        - return services (IServiceCollection)
    - Controllers/AccountControllers.cs = gère les routes de login et retourne un token
        - public AccountController(DataContext context, ITokenService tokenService) = _tokenService = tokenService;
        - route "register" 
        - return DTO : Token = _tokenService.CreateToken(user)
    - Interfaces/IdentityServiceExtensions.cs = gère l'identification des requêtes par les tokens et le token_key
    - Controllers/UserController.cs = gère les GET protégée par les [Authorize]



### client (Angular)

src/app: 
    - app-component.html : source app-component.ts (title, users) pour affichage
    - app-component.ts : route, init, stoque les datas reçues
    - app-modules.ts : modules dispo dans l'app pour chaque component

ssl : server/crt, server.key (ajouter sur la machine pour le browser), update de angular.json
    "options": {
            "sslKey": "./ssl/server.key",
            "sslCert": "./ssl/server.crt",
            "ssl": true

ng g -h
cd src/app : creer component dans l'app folder
ng g c nav --skip-tests
CREATE src/app/nav/nav.component.html (18 bytes)
CREATE src/app/nav/nav.component.ts (263 bytes)
CREATE src/app/nav/nav.component.css (0 bytes)
UPDATE src/app/app.module.ts (648 bytes) : update app-module "import { NavComponent } from './nav/nav.component';"

integrer un component dans un autre component: component selector app-nav => <app-nav></app-nav>, app-root => <app-root></app-root>

2 ways binding data :
    - data or propertie sin comonent => on les display dans les templates html
    - data from forms in html => recupere dans les componets
    - [] receive from component, () send to component from html, [()] 2_way binding

services shared among all comonents, grouped in _services:
    - ng g s account --skip-tests : CREATE src/app/_services/account.service.ts (136 bytes)
    - Injectable : service can be injected into service or component in the application
    - singleton
    - envoie les données du formulaire au serveur API



Observable ???
dans account.service.ts on crée l'observable sur le user reçu dans la méthode login 
private currentUserSource = new ReplaySubject<User>(1);
currentUser$ = this.currentUserSource.asObservable();
this.currentUserSource.next(user);

on s'abonne a currentUser$ dans navComponent.ts pour savoir si le user est dans le local storage
on rend public le account service dans navComponent.ts pour l'appeler dans navComponent.html

Dans app.component.ts, on utilise le local storage pour mettre le cuurentUser$ du account.servivr à jour si l'utilisateur s'est déjà connecté
    => so on refresh, le user est toujours loggé

ng g c home --skip-tests
CREATE src/app/home/home.component.html (19 bytes)
CREATE src/app/home/home.component.ts (267 bytes)
CREATE src/app/home/home.component.css (0 bytes) 
UPDATE src/app/app.module.ts (873 bytes) 
    => structure for home component


home component is a child of app.component : pass data from parent to child