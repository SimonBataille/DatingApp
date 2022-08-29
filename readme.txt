WiFi4IDT2020

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



### section 5 

client (Angular)
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
    - data or properties in component => on les display dans les templates html
- data from forms in html => recupere dans les components
    - [] receive from component, () send to component from html, [()] 2_way binding

services shared among all components, grouped in _services:
    - ng g s account --skip-tests : CREATE src/app/_services/account.service.ts (136 bytes)
    - Injectable : service can be injected into service or component in the application
    - singleton
    - envoie les données du formulaire au serveur API



Observable ???
dans account.service.ts on crée l'observable sur le user reçu dans la méthode login 
private currentUserSource = new ReplaySubject<User>(1);
currentUser$ = this.currentUserSource.asObservable();
this.currentUserSource.next(user); : mis à jour de l'observable dans la req login

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


home component is a child of app.component : pass data from parent to child :
    - parent = home.component.ts : this.http.get('https://localhost:5001/api/users').subscribe(users => this.users = users);
        - shorthand: users in the response body from API
        - assign this.users to users from users inside API response
        - pour users = List<AppUser>
    - child = register.component.ts : @Input() usersFromHomeComponent: any;
        - register.component.html : <option *ngFor="let user of usersFromHomeComponent" [value]="user.userName">

pass data from child to  parent
- registercomponent
    - <button class="btn btn-default mr-2" (click)="cancel()" type="button">Cancel</button>
    - @Output() cancelRegister = new EventEmitter();
    - cancel() { this.cancelRegister.emit(false); }
- home comonent.ts
    - cancelRegisterMode(event: boolean) {
        this.registerMode = event;
    }
    - <app-register [usersFromHomeComponent] = "users" (cancelRegister)="cancelRegisterMode($event)"></app-register>

  constructor(private accountService: AccountService) { } : injecte accountService inside the component


SECTION 6 angular routing:
- route components not pages like in trad html
- ng g c member-list --skip-tests, ng g c messages --skip-tests, ng g c lists --skip-tests
- app-routing.modules.ts => impoit in app.modules.ts, array of routes
- router-oulet in app-component
- routre-link in nav.component

- constructor(public accountService: AccountService, private router: Router) {} : import le service router dans un component

- notify the user
    - cd client
    - npm install ngx-toastr@13.0.0 : install le package toastr for angular dans client
    - style for toast in angular.json
    - dans app.module : import { ToastrModule } from 'ngx-toastr';
            ToastrModule.forRoot({
                positionClass: 'toast-bottom-right'
            }),
    - now we have a toastr service inside app/navComponent: import { ToastrService } from 'ngx-toastr';  constructor(public accountService: AccountService, private router: Router, private toastr: ToastrService) {}
    -toastr launch pop-up when error !!

-  *ngIf="accountService.currentUser$ | async" : check if i am authenticated

- route gard : cd _gards, ng g guard auth --skip-tests, interface canActivate, CREATE src/app/_gards/auth.guard.ts (457 bytes)
    - a class can activate the interface guard
    - account.service: currentUserSource observable by any class or service = the guards can subscribe to observable Automatically !!!
    - pipe because we don't need to subscribe
    - app.routing:  {path: 'members', component: MemberListComponent, canActivate: [AuthGuard]}, = guard sur la route members si pasconnecter
    - auth.guard: canActivate ()this.toastr.error('You shall not pass!')

    - children cover covered by authguard
    - ng-container to apply *ngIf="accountService.currentUser$ | async" => don't produce html

- a boostrap theme
    - cd client
    - npm install bootswatch@4.5.2: available in our app
    - angular.json

- keep app.module tidy (we are going to use more and more bootstrap components)
     - cd src/app/_modules
     - ng g m shared --flat: CREATE src/app/_models/shared.module.ts (192 bytes)
     - shared.module : toast, bsdropdown, export: []
     - app.module : import SharedModule


### Section 7 : error handling 
- api middleware
- angular interceptor: req going out and resp coming back
- exceptions

API
    - specific controller in API: buggy to provoke errors
        - ActionResult<string> : core.mvc
            - routes: auth, not-found, server-error, bad-request
        - register.dto : [StringLength(8, MinimumLength = 4)] password lenght error route account/rgister

    - startup : app.UseDeveloperExceptionPage(); dans le middleware container

    - Error folder:
            - ApiException : for every error in our API

    - middleware folder : 
        - middleware to handle exception, reqDelegate, HttpContext = error happen in http context req
        - exception are generated from controller with function from Microsoft.AspNetCore.Mvc (http) and remonte up back to the middleware wich use function from Microsoft.AspNetCore.Mvc (http)
        - startup : utilise notre middleware pour traiter les exceptions
                // if (env.IsDevelopment())
                // {
                //     app.UseDeveloperExceptionPage();
                //     app.UseSwagger();
                //     app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "WebAPIv5 v1"));
                // }

                app.UseMiddleware<ExceptionMiddleware>(); => top of the middleware
                - passe la req endessous de lui, si une exception est déclenchée, elle remonte puis elle prise en charge par ExceptonMiddleware

                les middleware sont liés les uns aux autres dans startup

CLIENT
    - mkdir src/app/errors
    - ng g c test-errors --skip-tests (g c : generate component)
        CREATE src/app/errors/test-errors/test-errors.component.html (26 bytes)
        CREATE src/app/errors/test-errors/test-errors.component.ts (294 bytes)
        CREATE src/app/errors/test-errors/test-errors.component.css (0 bytes) 
        UPDATE src/app/app.module.ts (1548 bytes)
    - test errors response from api in test-errors.component.ts : route to send bad req to buggy api
    - boutton pour envoyer les req dans test-errors.component.html
    - ajoute les routes dans app-routing.modules.ts 

    - http interceptor : intercept globally exception in angular from API
        - mkdir src/app/_interceptors
        - ng g interceptor --skip_tests
            CREATE src/app/_interceptors/error.interceptor.ts (410 bytes)
        - intercept req out or resp back
        - constructor(private router: Router, private toastr: ToastrService) {}, router to redirect, toast to notify
        - intercept ret observable ==> use pipe !! 
        - provide interceptor dans les modules dans app.modules.ts, on ajoute l'interceptor à celui déjà présent dans angular
        - flat() method, tsconfig.ts, "es2019",

    - 404 handler : cd app/errors, ng g c not-found --skip-tests

    - 500 server error: cd app/errors, ng g c server-error --skip-tests
            CREATE src/app/errors/server-error/server-error.component.html (27 bytes)
            CREATE src/app/errors/server-error/server-error.component.ts (298 bytes)
            CREATE src/app/errors/server-error/server-error.component.css (0 bytes)
            UPDATE src/app/app.module.ts (1924 bytes)

        - case 500 dans error.interceptor.ts, component interceptor is load when error from API
            - dans server-error.component.ts
                constructor(private router: Router) {
                    const navigation = this.router.getCurrentNavigation();
                    this.error = navigation?extras?.state?.error;
                }
                constructor only called one time !!! disapear after refresh
            - 1. error is intercepted by interceptor
            - 2. interceptor route the error to /server-error
            - 3. la route /server-error is redirect to server-error.component by app-routing.modules
            - 4. le server-error.component recupere l'error dans son constructeur
            - 5. on affiche le html du server-error.component

summary : 
    - API : create own exception class, exception handling middleware => return our own exceptions to the client rather than utilisize the exception middleware provided by .NET cover
        - middleware for dev and prod
    - Angular : interceptor => check any errors from API response

SECTION 8 : extend API, add functionnality, Entity Framework Relationships, EF Conventions, seed databases, repository pattern, automapper

- extend entities: photo.cs
- datetime Extensions
- [Table("Photos")], using System.ComponentModel.DataAnnotations.Schema; in Entities/photo.cs => create the table photos
    - cd API
    - dotnet ef migrations add ExtendedUserEntity
        - new migrations with photos and foreign key !!!
        - attention nullable id user, on delete behaviour => should delete the photos
        - we have to use entity framework convention : dotnet ef migrations remove
    - fully define relatonship: AppUser, AppUserId in photo => dotnet ef migrations add ExtendedUserEntity => dotnet ef database update
- seed data: lazy efficient way : JSON data, json generator => UserSeedData.json
-seed.cs class : tracking (context.Users.Add(user)), sync with database: (await context.SaveChangesAsync())
    - call Program.cs => before application starts
        - try/catch: we are outside our middleware !!
        - await context.Database.MigrateAsync(); : restart app to apply migrtions
        - remove Run() => call it after !!!
- sending data :
    - dotnet ef database drop
    - dotnet wtch run => all data populated
- [Authorize] pour toute la classe UserController

- repository pattern : redesign architecture pattern app
    - repository between the domain and data mapping layers acting like an in-memry domain object collection
    - web server (kestrel server) in API, req comes into controller endpoint, we are injecting dbContxt in our controller,
      dbContext represents a session with our database, dbcontext is translating the logic, the queries that we are writing in controller
      and fishing the data from db and return it to the controller which return to the CLIENT
    - repository pattern : a layer of abstarction, controller uses repo and executes methods inside there
                         repository uses the dbcontext to go and executes the logic
    - why :
        - encapsulate the logic : lots of methods inside DbContext<DbSet>, few in repository, only a few method for the controller
        - reduce duplicate query logic in controller : create logic in controller, and execute in repository
        - promote testability
        - repository have interfaces and implementation class for repository => repo iface in controller, and implementation class
        - repo pattern decentralize our query
        - ~ entityframework abstarction of database, repo is abstarction of entityframework

- iface and implementation for user: Interfaces/IUserRepository, only provide method for entity
    - Data/UserRepository for implementing
    - ApplicationServiceExtensions : add service for the repository

- what to update in controller to use repo
    - UsersController.cs : IUserRepository in spite of DatatContext

- MemberDto : avoid circular object reference => transfer photo_dto and members_dto

- automapper: nugget = automapper
    - helpers class
    - map one object to another
    - services.AddAutoMapper(typeof(AutoMapperProfiles).Assembly);

- create of automapper :
    - map from app_user entity to member_dto
    - map from photo entity to photo_dto
    - add auto_mapper to application_service_extension : services.AddAutoMapper(typeof(AutoMapperProfiles).Assembly);

- use of automapper :
    - change appuser to memberdto in usercontroller
    - inject Imapper iface in usercontroller
    - map appuser to memberdto : return usertoreturn = _mapper.Map<MemberDto>(user);

- configure automapper : photourl
    - automapperprofile : .ForMember(dest => dest.PhotoUrl, opt => opt.MapFrom(src =>
                    src.Photos.FirstOrDefault(x => x.IsMain).Url))
    - linq expression !!!
    - is it optimal ?

- automapper queryable extension : premature optimization is the root of all evil
    - what we are doing in user_repository: get_user_by_username
        - get user from db
        - include photo collection
        - pass to the controller
    - inside controller
        - we have got entity in memory : getuserbyusernameasync
        - go in memory and map from one object to another
    - better to go and get thez properties we need from db and passes back a dto from database level
    - rather then get an entity and then converting into dto
    - create new method inside IUserRepository
        - Task<IEnumerable<MemberDto>> GetMembersAsync();
        - Task<MemberDto> GetMemberAsync(string username);
        - we are returning MemberDto in spite of AppUser
    - implement method inside UserRepository
        - if we didn't use automapper :
            - return await _context.Users
                .Where(x => x.UserName = username)
                .Select(user => new MemberDto{
                    Id = user.Id;
                    Username = user.UserName
                }).SingleOrDefaultAsync() = execute query to db
            - we select all the property !! need to use ProjectTo<MemberDto>
        - with automapper :
            - inject IMapper in UserRepository
            - public async Task<IEnumerable<MemberDto>> GetMembersAsync()
            {
                return await _context.Users
                    .ProjectTo<MemberDto>(_mapper.ConfigurationProvider)
                    .ToListAsync();
            }
            - in usercontroller 
            [HttpGet("{username}")]
            public async Task<ActionResult<MemberDto>> GetUser(string username)
            {
                var user = await _userRepository.GetMemberAsync(username);

                return await _userRepository.GetMemberAsync(username);
            }
            - but GetAge method : mapper had to get all user 
            - in AutoMapperProfiles
             .ForMember(dest => dest.Age, opt => opt.MapFrom(src => 
             src.DateOfBirth.CalculateAge()));
            - we optimize our request !!!

    Le usercontroler call getmemberAsync in spite of getUserAsync 






