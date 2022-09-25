Different types of configuring components selector.

by Element : <app-start><app-start/>, selector : "app-start"
by attribute : <div app-start><div/>, selector : "[app-start]"
by attribute : <div class='app-start'><div/>, selector : ".app-start"

---

---

DATA BINDING

1.STRING INTERPOLATION - {{}}
2.PROPERTY BINDING - innerText="{{language}}" or [innerText]="language" or [innerText]="'Javascript'"
3.EVENT BINDING
4.TWO WAY BINDING -

<input type="text" [(ngModel)]="language">

  <h2>{{language}}</h2>

in above example as input changes h2 also changes

########################################################################################################################

4.BINDING TO CUSTOM PROPERTIES

passing data from parent to child component (binding to children's property)

<app-one [languageName]="language" [frameworks]="messages"></app-one>

in children comp

@Input() frameworks: string[] | undefined;
@Input('languageName') language: string | undefined;

########################################################################################################################

5.BINDING TO AN CUSTOM EVENTS

    <app-one (languageChanged)="onChange($event)"></app-one>

    @Output() languageChanged = new EventEmitter<string>();

########################################################################################################################

6.PASSING LOCAL REFERENCES

    <h1 #welcome>hii</h1>
    <button type="button" (click)="changed(welcome)">Change</button>

    in local reference we pass entire html element
    we can't have access to local refs inside typescript file we can access then inside only templates.
    for accessing localReferences or components inside respective view we can use ViewChild decorator

    @ViewChild('serverContentInput', {static: true}) serverContentInput: ElementRef;
    or
    @ViewChild(oneComponent, {static: true}) component: ElementRef; //for getting access of the first occurrence oneComponent

    and then we have serverContentInput.nativeElement property

########################################################################################################################

7. Projecting Content into Components with ng-content

   by default angular will ignore all content placed between opening and closing tags of yor component
   like, <app-one>hii there!</app-one>, here hii there will not be printed.

   but we can use ngContent _directive_ inside appOne template file and print hii there! -
   <ng-content></ng-content>

   for accessing content inside ng-content passed by parent

   @ViewChild('serverContentInput', {static: true}) serverContentInput: ElementRef;

---

---

Angular Directive :
Directives are classes that add additional behavior to elements in your Angular applications.
Directives are instructions in the DOM

STRUCTURAL DIRECTIVES

    ngFor and ngIf

behind the scenes :

    <div *ngIf="!onlyOdd">
          <li
            class="list-group-item"
            [ngClass]="{odd: even % 2 !== 0}"
            [ngStyle]="{backgroundColor: even % 2 !== 0 ? 'yellow' : 'transparent'}"
            *ngFor="let even of evenNumbers">
            {{ even }}
          </li>
    </div>

above code transformed to =>

    <ng-template [ngIf]="!onlyOdd">
          <div>
            <li
              class="list-group-item"
              [ngClass]="{odd: even % 2 !== 0}"
              [ngStyle]="{backgroundColor: even % 2 !== 0 ? 'yellow' : 'transparent'}"
              *ngFor="let even of evenNumbers">
              {{ even }}
            </li>
          </div>
    </ng-template>

########################################################################################################################

ATTRIBUTE DIRECTIVES

ngClass and ngStyle

    [ngClass] = "{even:number%2===0}"
    [ngStyle] = "{backgroundColor : number%2===0 ? 'red' : 'yellow'}"

########################################################################################################################

CUSTOM DIRECTIVE

we can get acccess to element directive seats on via -

    this.elementRef.nativeElement.style.backgroundColor = 'green';
      or

when we have not access to DOM a better approach will be -
this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'blue');
or
@HostBinding('style.backgroundColor') backgroundColor: string;

with host listeners we can listen to events on elements on which directive seats -
@HostListener('mouseenter') mouseover(eventData: Event) {
// this.renderer.setStyle(this.elRef.nativeElement, 'background-color', 'blue');
this.backgroundColor = this.highlightColor;
}

########################################################################################################################

BINDING TO DIRECTIVE PROPERTIES

    <p [appBetterHighlight]="'red'" defaultColor="yellow">Style me with a better directive!</p>
    or
    <p appBetterHighlight="red" defaultColor="yellow">Style me with a better directive!</p>

    inside directive -

    @Input('appBetterHighlight') highlightColor: string = 'blue';
    @Input() defaultColor: string = 'transparent';

---

---

SERVICES

Angular services are singleton objects that get instantiated only once during the lifetime of an application.
The main objective of a service is to organize and share business logic, models, or data and functions with different components
of an Angular application.

i) if we provide a service in app module then same instance of service is available application wide.
ii) if we provide a service in app component then same instance of service is available for all
components but not for other services.
iii) if we provide a service in any other component then same instance of service is available for that component
and all of it's children components.

- if we provide same service in the child component as provided in the parent component then we override the parent
  service instance.(instances are different)

we can provide a service to specific component by adding it to components providers array.
@component({
providers:[authService]
})

or

@component({
providers:[authService]
})

---

---

DI (Dependency injection in Angular)

Dependencies are services or objects that a class needs to perform its function. Dependency injection, or DI, is a design pattern in which
a class requests dependencies from external sources rather than creating them.

    import { Injectable } from '@angular/core';
    import { HEROES } from './mock-heroes';

    @Injectable({
      // declares that this service should be created
      // by the root application injector.
      providedIn: 'root',
    })
    export class HeroService {
      getHeroes() { return HEROES; }
    }

---

---

ROUTER

navigating with router links -

    <a routerLink='/' [routerLinkActive] = "'active'" [routerLinkActiveOptions] = "{ exact:true }" >Home</a>
    <a routerLink='/user/signIn'>Log In</a>
    <a [routerLink]="['/user','signIn']">Log In</a>

programatic navigation -

      router.navigate(['/signIn']);
      router.navigate(['../'],{ relativeTo : this.route });

getting route params -

      this.userId = this.route.snapshot.params['id];

fetching route params reactively -

    above approach is usefull when reaching corresponding route creates new component
    every time that is if we are on the component and we request router to load the present component by changing route
    params then Angular will not create new component, in that case we need to look for possible changes that may occure =>


      this.route.params.subscribe(
        params:Params => this.userId = params['id']
      );

Passing Query Parameters and Fragments -

      router.navigate(['auth'],{ queryParams:{allowEdit:true}, fragment:'loading'});

Preserve Query Parameters -

      router.navigate(['edit'],{ queryParamsHandling:'preserve' });

Auth Guards -

      The canActivate method returns a boolean indicating whether or not navigation to a route should be allowed.

        canActivate(route: ActivatedRouteSnapshot,
              state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
          return this.authService.isAuthenticated()
          .then(
            (authenticated: boolean) => {
              if (authenticated) {
                return true;
              } else {
                this.router.navigate(['/']);
              }
            });
          }

          canActivateChild(route: ActivatedRouteSnapshot,
                   state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
            return this.canActivate(route, state);
          }

RouterState -

    Represents the state of the router as a tree of activated routes.

      const state: RouterState = router.routerState;
      const root: ActivatedRoute = state.root;
      const child = root.firstChild;
      const id: Observable<string> = child.params.map(p => p.id);

Passing static data to a route -

       {
         path: 'not-found',
         component: ErrorPageComponent,
         data: { message: 'Page not found!' },
       }

    retriving -

        this.route.data.subscribe(
          data=>{}
        )

Resolving dynamic data with resolve guard -

    {
        path: ':id',
        component: ServerComponent,
        resolve: { server: ServerResolver },
    },

      export class ServerResolver implements Resolve<Server> {
      constructor(private serversService: ServersService) {}

      resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Server> | Promise<Server> | Server {
      return this.serversService.getServer(+route.params['id']);
    }

}

this.route.data
.subscribe(
(data: Data) => {
this.server = data['server'];
console.log(data);

        }
      );

---

---

OBSERVABLE

An Observable is a lazily evaluated computation that can synchronously or asynchronously return zero
to (potentially) infinite values from the time it's invoked onwards.
Angular makes use of observables as an interface to handle a variety of common asynchronous operations.

      this.firstObsSubscription = interval(1000).subscribe(count => {
       console.log(count);
      });

    const customIntervalObservable = new Observable((subscriber) => {
      let count = 0;
      setInterval(() => {
        subscriber.next(count);
        if (count === 5) {
          subscriber.complete();
        }
        if (count > 3) {
          subscriber.error(new Error('Count is greater 3!'));
        }
        count++;
      }, 1000);
    });

    this.firstObsSubscription = customIntervalObservable
      .pipe(
        filter((data) => {
          return data > 0;
        }),
        map((data: number) => {
          return 'Round: ' + (data + 1);
        })
      )
      .subscribe(         // An Observer is a consumer of values delivered by an Observable. % observable.subscribe(observer); %
                         //Observers are just objects with three callbacks, one for each type of notification that an Observable may deliver.
        (data) => {
          console.log(data);
        },
        (error) => {
          console.log(error);
          alert(error.message);
        },
        () => {
          console.log('Completed!');
        }
      );

When you subscribe, you get back a Subscription, which represents the ongoing execution. Just call unsubscribe() to cancel the execution.

- func.call() means "give me one value synchronously"
- observable.subscribe() means "give me any amount of values, either synchronously or asynchronously"

Subject -

An RxJS Subject is a special type of Observable that allows values to be multicasted to many Observers.
While plain Observables are unicast (each subscribed Observer owns an independent execution of the Observable), Subjects are multicast.
A Subject is like an Observable, but can multicast to many Observers. Subjects are like EventEmitters: they maintain a registry of many listeners.

    const subject = new Subject<type>();

plain subject not gives any initial value or current value at the time of subscribing to it, it gives valuef when we subscribe to it and then emit a value.

The BehaviorSubject has the characteristic that it stores the “current” value. This means that you can always directly get the last emitted(or initial) value from the BehaviorSubject.

---

---

HTTP

post request :

    post<T>(url: string, body: any | null, options?: {
        headers?: HttpHeaders | {
            [header: string]: string | string[];
        };
        observe?: 'body';
        params?: HttpParams | {
            [param: string]: string | string[];
        };
        reportProgress?: boolean;
        responseType?: 'json';
        withCredentials?: boolean;
    }): Observable<T>;

---

---

MODULES (NgModule)

App module -
Angular analyzes NgModules to understand your application and it's features
Angular module all the building blocks your app uses: Components, Directives, services
An app requires at least one module(AppModule) but it can be split into multiple modules
Core angular features are included in Angular modules(eg. FormsModule) to load them when needed
You can't use a feature/building block without including it into a Module.

declarations : it consists of declarations of all the pipes, directives and component your app needs.

Imports array : Imports array allows us to import other modules into current module.

Providers array : Any service that you plan on injecting(applications wide) needs to be provided at here.

Bootstrap array : Defines which Components is available right into that index.html file

Entry components : Components which will be created dynamically(programmatic) needs to included here.

DI (dependency injection)

Dependency injection simply means that all instances of needed dependencies to construct an object is passed to a object's constructor.

    class Engine {
      startEngine(){
        console.log("brrrrrr")
      }
    }

    class Car {
      constructor(private engine:Engine){}
    }

    main() {
      const engine = new Engine();
      const car = new Car(engine)
    }

Similarly we provide a service to the component's constructor, and then we add that dependency to the components
providers array(or we can provide service globally).

@Component({
selector: ...,
template: ...,
providers:[ MyService ]
})
