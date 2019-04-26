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

<img src="/assets/categories/angular/dependency-injection/local-singleton.svg" alt="Illustrates Dependency Injection Lookups" title="Illustrates Dependency Injection Lookups">

#### Affect DI Lookups

- Optional
- SkipSelf
- Host

## Global Singletons
### Services + ProvideIn Root
### Services + Providers Array Module

## Local Singletons

### Components & Directives

## Semi Global (Lazy Loading) / Pitfalls / forRoot & forChild

### forRoot
### forChild

## Implementations
### useClass
### useValue
### useFactory

## Injection Tokens
### Multi Injection Tokens

## Use Cases
### Local Configuration
### Lazy Configuration
