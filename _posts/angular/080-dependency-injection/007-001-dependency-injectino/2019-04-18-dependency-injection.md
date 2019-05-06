---
title: "Angular Dependency Injection"
description: "Angular Dependency Injection"
author: "Peter Müller"
published_at: 2019-04-18 21:25:02.000000Z
header_source: https://unsplash.com/photos/EhLH-WN7F7I
categories: angular
chapter: Angular Dependency Injection
chapter_sort: 7.10
---

## Angular Dependency Injection

Angular provides a powerful dependency injection system, short DI. Dependency Injection brings a lot of benefits. It keeps other classes, such as components, directives or modules, clean since we don't have to care about how the services must be instantiated. The code is easier to test because the implementation and mock objects can be replaced easily.  
Dependency Injection is also called **inversion of control** because **the injector has control** over service instantiation.

To understand how DI works under the hood we have to figure out how an angular application is structured. An Angular app can be seen as split into an TypeScript part and the view which represents the HTML sturcture.

The image below illustrates a simple application which consists of three modules and some components which provides a navigation, a header and main content with some articles.

<img src="/assets/categories/angular/dependency-injection/basics.svg" alt="Illustrates Dependency Injection Basics" title="Illustrates Dependency Injection Basics">

> **Hint:** services can have dependencies, too. Watch out for dependency cycles even the build tools will warn about circular dependencies.

### Basic Dependency Injection with Services

The code-snippet below illustrates a simple service which does literally nothing. To make the class injectable it must be decorated with the `@Injectable` decorator which is located in the `@angular/core` module.

When using the Angular CLI the service is configured with `provideIn: 'root'` by default. This tells Angular to instantiate the service with a **global singleton** which will be shared across the application. Hence the service instance will be reused.

```ts
// my.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  provideIn: 'root'
})
export class MyService {}
```

As in the previous chapter [Angular Services](/angular/services) mentioned the injection is based on the **type of the class.** To inject the service it must be simply passed to the constructor of another Angular service, component, directive or any other class which is **instantiated by Angular.**

```ts
// my.component.ts

@Component({ /* ... */ })
export class MyComponent {
  constructor(private myService: MyService) {}
}
```

>**Hint:** is not possible to use Angulars dependency injection when creating classes manually using the new keyword: `new MyService(/* Huh? */)`

### Injectors and Lookups

In Angular there are two kinds of injectors: a **RootInjector** and a **ComponentInjector**. That means that services can be provided in root level which is the default and most common way and on component level.

Usually services are injected into components, directives or pipes which belong to the view. First Angular checks if the service has been provided by the current component injector. If the requested service is not provided Angular looks up for a provide instance in the parent component and so on. DI lookups ends up in the AppComponent which is the root of our component tree. If no instance could be determined then Angular checks if a global instance of the service is provided. If no service could be found an error will be thrown.

The image below illustrates the lookup for `AService` which is provided globally since the class is decorated and configured with `provideIn: 'root'`.

<img src="/assets/categories/angular/dependency-injection/lookup.svg" alt="Illustrates Dependency Injection Lookups" title="Illustrates Dependency Injection Lookups">

> **Hint:** since directives are kind of components but without a template you can provide services here, too.

#### Local Singletons

In Angular it is possible to provide dependencies on component level. The `@Component` decorator accepts an `providers` property which defines an array with dependencies such as services. Services which are provided in components or directives are also known as `Local Singletons`. That means that Angular uses the ComponentInjector of those service and creates the instance. The created service is now avaialble on the provided component and all children. If the service is used in the related component or in a child, the DI lookup will stop on the providing component.

<img src="/assets/categories/angular/dependency-injection/local-singleton.svg" alt="Illustrates Dependency Injection Lookups" title="Illustrates Dependency Injection Lookups">

#### Affect DI Lookups

DI lookups always follow the same way from bottom to top, starting from the component, directive or pipe which injects the service. Angular comes with some decorators which helps affect how DI lookups work.

##### @Optional

If DI lookups end up in root but cannot find a global provided service Angular will throw an error about the missing dependency. However it is possible to mark the dependency as optional. Angular will now throw an error and the constructor argument will be `null` instead of having the service instance.

```ts
class MyComponent {
  constructor (@Optional() a: AService) {
    if (a) {
      // AService is provided somewhere in the application
      console.log(a);
    }
  }
}
```

> **Hint:** when using the `@Optional` decorator it is not possible to declare the argument as `private`, `public` or `protected` to make it implicit a class property. This have to be implemented manually.

##### @SkipSelf

Angular dependency injection lookups start at the component where the service will be injected itself. If so, the instance will be created at the same component and DI stops immediately. `@SkipSelf` decorator instructs Angular to skip the lowest component and start looking up from the parent component up.

```ts
class MyComponent {
  constructor (@SkipSelf() a: AService) {

  }
}
```

When using SkipSelf the chance to find a service decreases a bit. It is possible to combine `@Optional` and `@SkipSelf`.

```ts
class MyComponent {
  constructor (@Optional() @SkipSelf() a: AService) {
    if (a) {
      // The service is provided in one of the parent components or globally
    }
  }
}
```

##### @Host

The `@Host` decorator forces angular to look the next parent component injector and **do not lookup further.**

Components literally 'host' a view with elements which may contain directives or pipes. Since we can inject services into pipes and directives, too, the related host is the containing component.

The combination of `@Host` with `@Optional` is recommended.

```html
<my-component> <!-- Host Component -->
  <h1 [myDirective]="anyValue | myPipe"> <!-- View / Child Element -->
    I have a directive and pipe
  </h1>
</my-component>
```

```ts
// This is the host component
@Component({
  selector: 'my-component'
  providers: [AService],
  template: `<h1 [myDirective]="anyValue | myPipe">I have a directive and pipe</h1>`
})
class MyComponent {
  anyValue = 'Hello!';
}

// Child / view directive
@Directive({
  selector: '[myDirective]'
})
class MyDirective {
  constructor(@Optional() @Host() a: AService) {}
}

// Child / view pipe
@Pipe({
  name: 'myPipe'
})
class MyPipe {
  constructor(@Optional() @Host() a: AService) {}
}
```

## Instantiation & Implementation

In Angular it is possible to enhance the implementation and instantiation of services. Before proceed we'll have a short recap on how services can be provided.

**1. `provideIn`**

Using `provideIn: 'root'` the service becomes **global**. The benefit here is that this service is `tree-shakable` which means that the code will be removed on build-time if the service is not used in the application. [Read more about Tree shaking](https://en.wikipedia.org/wiki/Tree_shaking)  and dead code elimination.

```ts
@Injectable({
  provideIn: 'root'
})
export class MyService {}
```

**2. `providers` on NgModule level**

Services can be provided in an Angular module which makes the service **global**, too.

```ts
@NgModule({
  providers: [MyService]
})
export class MyModule {}
```

**3. `providers` on component level**

This makes to service to a **local** singleton.

```ts
@Component({
  // ...
  providers: [MyService]
})
export class MyComponent {}
```

### Configuration and shorthands

Declaring service classes in the providers array are shorthands. In Angular it is possible to change the service implementation using `useClass`. The example below illustrates how Angular handles shorthands.

```ts
{
  providers: [MyService]
}

// Above is the shorthand for the following configuration

{
  providers: [{
    provide: MyService,
    useClass: MyService
  }]
}
```

### useClass

With `useClass` we can tell Angular which class should be instantiated instead of the requested one. This is useful if we have different implementations of a class but with the same purpose.

```ts
@Injectable()
export abstract class DataService  {
  public abstract getData();
}

@Injectable()
export class HttpDataService extends DataService {
  public getData() {
    return this.http.get('/data');
  }
}

@Injectable()
export class DummyDataService extends DataService {
  public getData() {
    return of([]);
  }
}
```

> **Hint:** it is mandatory to use a class instead of an interface. Interfaces do not work.

```ts
@Component({
  selector: 'my-component',
  styles: ``,
  template: ``,
  providers: [{
    provide: DataService,
    useClass: HttpDataService
  }]
})
export class MyComponent {
  constructor(private dataService: DataService) {}
}
```

### useValue

It is even possible to replace service classes with any other value. The use of `useValue` in production code is fairly rare and should **used with caution** since the type interface of the class might not match with the actual value. It is more useful for mocking object instances in unit tests.

```ts
@Component({
  selector: 'my-component',
  styles: ``,
  template: ``,
  providers: [{
    provide: DataService,
    useValue: {
      getData() {}
    }
  }]
})
export class MyComponent {
  constructor(private dataService: DataService) {}
}
```

### useExisting

Sometimes it makes sense to map a class to an existing instance. For this purpose the property `useExisting` comes into the game.

```ts
providers: [
  // This class will be instantiated by the injector
  ExistingService,
  {
    provide: AnotherService, // This class refers to ExistingService
    useExisting: ExistingService
  }]
```

`useExisting` can end end in confusion since TypeScript provides code completion for `AnotherService` but on runtime `ExistingService` will be accessed. There are just a few but powerful pattern for aliasing services.

One aproach is to slice a large services and only provide necessary parts which are actually needed. In the example below we have a HTTP service which provides a lot request methods. The second service provides only a subset.

```ts
export class HttpService {
  get() {}
  put() {}
  post() {}
  patch() {}
  delete() {}
  options() {}
}

export class SlicedService {
  get() {};
  post() {};
}

providers: [
  HttpService,
  {
    provide: SlicedService,
    useExisting: HttpService
  }
]
```

> **Hint:** please note that both methods should be defined as in the aliased service for proper code completion and to avoid unexpected problems on runtime.

### useFactory

Factories bring a lot of flexibility when it comes to dynamic instantiaten of services or values. The `useFactory` property defines a function which returns the result value which can be anything.

A factory could be used to inject a third-party class which is no qualified Angular service. The following code snippet will illustrate how to make a non-Angular service injectable.

```ts
providers: [{
  provide: MyAngularService,
  useFactory() {
    return new OtherClass();
  }
}]
```

> **Hint:** the `useFactory` function will be only called once and the return value will be reused at further injections. This can be simply tested with a random value such as `Math.random()`. The value will always be the same.

It is also possible to use a `deps` array which defines all dependencies which are required to create the instance or value. The example below shows how to use a different service for each stage, development and production.

```ts
providers: [
  DevelopmentService,
  ProductionService,
  {
    provide: MyService,
    useFactory(dev: DevelopmentService, prod: ProductionService) {
      return environment.production ? prod : dev;
    },
    deps: [DevelopmentService, ProductionService]
  }
]
```

> **Hint:** all arguments in `useFactory` must have the same sequence as the dependencies declared in `deps`.

Dependencies can be marked as optional, too. To do so the dependency must be declared as an array. The Optional class is the same as the decorator mentioned in the DI lookup chapter.

```ts
import { Optional } from '@angular/core;

// ...

deps: [
  AService, 
  BService, 
  [new Optional(), CService]
]
```

## Injection Tokens

Creation of Angular service is quite flexible. We can provide ans service class and return any other value. Declaring classes to inject other values is possible but can result in confusion and unexpected behavior. Angular also supports so called `InjectionToken`s which can be injected like service but usually hold no service instances.

InjectionTokens also support TypeScript generics, to clearify which type can be expected when injecting those tokens.

```ts
// tokens.ts
import { InjectionToken } from '@angular/core';

export const API_URL = new InjectionToken<string>('API Url');
```

> **Hint:** It is a good practice to define tokens in a separate file to avoid circular dependencies.

```ts
providers: [{
  provide: API_URL,
  useValue: environment.apiUrl
}]
```

Injection tokens are just constants which are not decorated with `@Injectable` and have no information about their type and how to inject it. To inject those tokens, the `@Inject` decorator must be used. In the example below we are going to inject the API_URL token to an ApiService which makes HTTP calls on the given backend.

```ts
// api.service.ts
import { API_URL } from './tokens';

@Injectable({
  provideIn: 'root'
})
export class ApiService {
  private apiUrl: string;

  constructor (@Inject(API_URL) apiUrl: string) {
    this.apiUrl = apiUrl;
  }
}
```

> **Hint:** injection tokens are also a good way to hold configuration objects.

### Multi Injection Tokens

It is possible to provide multiple values for the same token. This requires the use of an array of the expected type. This technique is often used when multiple modules add further data or functionality to a single dependency.

For example we have an Angular module for each page like `/home` or `/disclaimer` and both want to register links to a global menu.

Just follow the small StackBlitz example to see how it works [https://stackblitz.com/edit/angular-multi-providers]()

```ts
// menu-link.interface.ts

export interface MenuLink {
  name: string;
  routerLink: string | any[];
}
```

```ts
// tokens.ts
// ...

export const MENU_LINKS = new InjectionToken<MenuLink[]>('Menu Links');
```

> **Hint:** please note that in this case we have an array of `MenuLink`s.

```ts
// legal.module.ts
// ...

providers: [{
    provide: MENU_LINKS,
    useValue: {
      name: 'Data Security',
      link: '/data-security'
    },
    multi: true
  }, {
    provide: MENU_LINKS,
    useValue: {
      name: 'Imprint',
      link: '/imprint'
    },
    multi: true
  }]
```

> **Hint:** Each menu entry must be provided separate. It is also mandatory to mark every item with `multi: true` otherwise Angular will thow an exception that multi and single providers can't be mixed.

```ts
// menu.component.ts
// ...

export class MenuComponent {
  menuLinks: MenuLink[] = [];

  constructor(@Inject(MENU_LINKS) menuLinks: MenuLink[]) {
    if (menuLinks) {
      this.menuLinks = menuLinks;
    }
  }
}
```
> **Hint:** keep in mind that `@Inject()` will return `null` if no menu link has been provided. MenuComponent has do deal with that situation.

## Lazy Loading / forRoot & forChild

To understand the following problem and how to solve it you have to know the basics of lazy loading in Angular which is done by routing. But for short: Angular provides a way to lazy-load modules when a specific route is requested. Then the build will split the application into multiple chunks. When the user requests the route the chunk will be loaded on demand.

The image below illustrates three modules. The `AppModule` which is the default entry point in our application and a `LazyModule` which will be loaded when the user hits `/lazy`. Both modules depends on `SharedModule` which provides functionality across the application. Furthermore App and Lazy consists of components which injects a `SharedService` which is provided in the SharedModule.

We already learned, that dependencies which are provided on module level are `global`. **But** when lazy loading comes into the game it get's a bit tricky and can result in unexpected problems.

<img src="/assets/categories/angular/dependency-injection/lazy-loading-pitfall.svg" alt="Illustrates Dependency Injection Lazy Loading Pitfalls" title="Illustrates Dependency Injection Lazy Loading Pitfalls">

When Angular builds the application and webpack detects a new entry point at `/lazy` it creates a new chunk. The picture below shows two chunks. Since both chunks relies on `SharedModule` it will be initialized for a second time when the user enters `/lazy`.

And that's the point where DI is a bit messy.

<img src="/assets/categories/angular/dependency-injection/lazy-loading-chunks.svg" alt="Illustrates Dependency Injection Lazy Loading Chunks" title="Illustrates Dependency Injection Lazy Loading Chunks">

As we already learned DI looks up on component level until it reaches module level. Everything which is provided in modules **should be global.** But when using lazy loading Angular creates **another global instance**. Now `AppComponent` and `LazyComponent` will retrieve **different services instances.** This may result in memory leaks and data inconsistency.

<img src="/assets/categories/angular/dependency-injection/lazy-loading-di-how-to.svg" alt="Illustrates Dependency Injection Lazy Loading How To" title="Illustrates Dependency Injection Lazy Loading How To">

> **Hint:** check out this [StackBlitz example](https://stackblitz.com/edit/angular-lazy-loading-pitfall) to see what happens.

### The `forRoot` & `forChild` pattern

If you already worked with Angular you might saw module imports using forRoot and forChild. This pattern is quite common and helps to avoid this issue.

Both methods are static and do not rely on a module instance. `forRoot` returns a module description and a providers array. This array consists of all "real" global dependencies. `forChild` returns a module description, too, but without providers.

This is pattern is relevant when using modules which are shared across the application. All shared modules have to be imported with forRoot on `AppModule` level. Any other module will use `forChild`. This guarantees that all services will be created **globally and only once.**

<img src="/assets/categories/angular/dependency-injection/lazy-loading-fix.svg" alt="Illustrates Dependency Injection Lazy Loading Fix" title="Illustrates Dependency Injection Lazy Loading Fix">

> **Hint:** whether you are using lazy loading, or not, using this pattern is a solid approach to avoid problems and makes it clear where dependencies are provided and injected. Check out this [StackBlitz example](https://stackblitz.com/edit/angular-lazy-loading-patch) to see how it works in production.
