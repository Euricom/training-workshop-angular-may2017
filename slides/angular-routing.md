# Angular
## Router & Navigation

<img src="./images/route66.jpeg" width="600px" /><br>
<small>by Peter Cosemans</small>

<small>
Copyright (c) 2017 Euricom nv.
</small>

<style type="text/css">
.reveal pre code {
    display: block;
    padding: 5px;
    overflow: auto;
    max-height: 800px;
    word-wrap: normal;
}
</style>

---

# The basics
> Simple routing

----

## Install & Setup

Install forms module

```bash
yarn add @angular/router
```

> Mark that the router has version: 3.x

----

## Setup

```ts
import { NgModule } from '@angular/core'
import { BrowserModule } from '@angular/platform-browser'
import { RouterModule, Routes } from '@angular/router'
import { AppComponent } from './app.component'

// create routes
const appRoutes: Routes = [
  { path: 'foo', component: FooComponent },
  { path: 'bar', component: BarComponent },
  { path: '', redirectTo: '/foo', pathMatch: 'full' },
  { path: '**', component: PageNotFoundComponent }
]

@NgModule({
    imports: [
        BrowserModule,
        RouterModule.forRoot(appRoutes),
    ],
    declarations: [
        AppComponent,
        FooComponent,
        BarComponent,
        PageNotFoundComponent,
    ],
    bootstrap: [ AppComponent ]
})
export class AppModule { }
```

----

## Components

FooComponent

```js
@Component({
    template: `
        <h1>Foo</h1>
    `
})
export class FooComponent() {}
```

BarComponent

```js
@Component({
    template: `
        <h1>Bar</h1>
    `
})
export class BarComponent() {}
```

PageNotFoundComponent

```js
@Component({
  template: `
    <h1>Page Not found</h1>
  `,
})
export class PageNotFoundComponent {
}
```

----

## Router-outlet & routerLink

Angular 2 routing expects you to have a base element in the head section

```html
<head>
    <base href="/">
    <!-- etc... -->
</head>
```

The router-outlet

```js
// app.component.ts
@Component({
    template: `
        <h1>My App</h1>
        <nav>
            <a routerLink="/foo">Foo</a>
            <a routerLink="/bar">Bar</a>
        </nav>
        <!-- Routed views go here -->
        <router-outlet></router-outlet>
    `
})
export class AppComponent() {}
```

----

## Styling

```js
@Component({
    template: `
        <a routerLink="/foo" routerLinkActive="router-active-link">Foo</a>
    `
    styles: [`
        .router-active-link {
            background-color: gray
        }
    `]
})
```

----

## Parameters

Declaring Route Parameters

```js
const appRoutes: Routes = [
    ...
    { path: 'foo/:id', component: FooComponent },
    ...
]
```

Linking to Routes with Parameters

```html
<a *ngFor="let product of products"
  [routerLink]="['/product-details', product.id]">
  {{ product.name }}
</a>
```

----

## Parameters

Get the current params

```ts
import { Component } from '@angular/core'
import { ActivatedRoute } from '@angular/router'

@Component({
    template: `
        <h1>Foo</h1>
        Parameter: {{id}}
    `
})
export class FooComponent() {
    id: String
    constructor(private route: ActivatedRoute) {}

    ngOnInit() {
        this.id = this.route.snapshot.params['id']
    }
}
```

----

## Parameters

Observe the route change

```ts
export class FooComponent() {
    contact: Contact
    private sub: any
    constructor(private route: ActivatedRoute) {}

    ngOnInit() {
        this.sub = this.route.params
            .map(params => params['id'])
            .switchMap(id => this.contactsService.getContact(id))
            .subscribe(contact => this.contact = contact)
    }
    ngOnDestroy() {
        this.sub.unsubscribe()
    }
}
```

----

## Route Change (by Code)

```ts
import { Location } from "@angular/common"
import { Router, ActivatedRoute } from "@angular/router"

constructor(private router: Router,
            private route: ActivatedRoute,
            private location: Location) {}
action() {
    // absolute
    this.router.navigate(['/about'])
    this.router.navigate(['/product-details', id])
    this.router.navigateByUrl(`/courses/${course.id}`)

    // relative
    this.router.navigate("../../parent", {relativeTo: this.route})
    this.router.navigate(["../../parent", {abc: 'xyz'}], {relativeTo: this.route})

    // back
    this.location.back()
}
```

----

### Child routes

```js
export const routes: Routes = [
  { path: '', redirectTo: 'product-list', pathMatch: 'full' },
  { path: 'product-list', component: ProductList },
  { path: 'product-details/:id', component: ProductDetails,
    children: [
      { path: '', redirectTo: 'overview', pathMatch: 'full' },
      { path: 'overview', component: Overview },
      { path: 'specs', component: Specs }
    ]
  }
]
```

---

# Navigation Guards
> Protecting your routes

----

## Guard Types

There are four different guard types we can use to protect our routes

* ***CanActivate*** - Decides if a route can be activated
* ***CanActivateChild*** - Decides if children routes of a route can be activated
* ***CanDeactivate*** - Decides if a route can be deactivated
* ***CanLoad*** - Decides if a module can be loaded lazily

----

## Can-Activate Guard

Create 'CanActivate' guard

```ts
import { Injectable } from '@angular/core'
import { CanActivate } from '@angular/router'

import { AuthService } from './auth.service'

@Injectable()
export class AuthGuard implements CanActivate {
    constructor(private authService: AuthService) {}

    // we need this function
    canActivate() {
        return this.authService.isLoggedIn()
    }
}
```

And use it on a route

```ts
{
    path: '',
    component: SomeComponent,
    canActivate: [AuthGuard]
}
```

Don't forget to register the guard in the module.

----

## Configure guard

Specify custom properties on route

```ts
{
    path: '',
    component: FooComponent,
    canActivate: [AuthGuard],
    data: { roles: ['super-admin', 'admin'] }
}
```

And use them in the guard

```ts
@Injectable()
export class AuthGuard implements CanActivate {
    constructor(private router: Router) {}

    canActivate(route: ActivatedRouteSnapshot,
                state: RouterStateSnapshot): boolean {
        let roles = route.data["roles"] as Array<string>
        if(this.authService.isInRole(roles)) {
            this.router.navigate(['/login'])
            return false
        }
        return true
    }
}
```

----

## Can-Deactivate Guard

```ts
import { CanDeactivate } from '@angular/router'
import { MyComponent } from './app/my.component'

export class ConfirmDeactivateGuard implements
                            CanDeactivate<MyComponent> {
    // we need this function
    canDeactivate(target: CanDeactivateComponent) {
        if(target.hasChanges()){
            return window.confirm('Do you really want to cancel?')
        }
        // return bool, promise or observable
        return true
    }
}
```

----

## Can-Deactivate Guard - Universal

```ts
import { Injectable } from '@angular/core'
import { CanDeactivate } from '@angular/router'
import { Observable } from 'rxjs/Observable'

export interface CanComponentDeactivate {
    canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean
}

@Injectable()
export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
    canDeactivate(component: CanComponentDeactivate) {
        return component.canDeactivate ? component.canDeactivate() : true
    }
}
```

Component

```ts
export class MyComponent implements CanComponentDeactivate {
    canDeactivate() {
        console.log('i am navigating away')
        // check and handle route change

        return true  // can deactivate
    }
}
```

---

# Route resolvers
> Get data before route change

----

## Resolve

Route resolvers allow us to provide the needed data for a route, before the route is activated.

```ts
export const AppRoutes: Routes = [
  ...
  {
    path: 'contact/:id',
    component: ContactsDetailComponent,
    resolve: {
      contact: ContactResolve
    }
  }
]
```

Contact Resolver

```ts
import { Injectable } from '@angular/core'
import { Resolve, ActivatedRouteSnapshot } from '@angular/router'
import { ContactsService } from './contacts.service'

@Injectable()
export class ContactResolve implements Resolve<Contact> {
    constructor(private contactsService: ContactsService) {}
    resolve(route: ActivatedRouteSnapshot) {
        // return observable or promise
        return this.contactsService.getContact(route.params['id'])
    }
}
```

----

## Resolve Data

```ts
@Component()
export class ContactsDetailComponent implements OnInit {
    contact: any
    constructor(private route: ActivatedRoute) {}
    ngOnInit() {
        this.contact = this.route.snapshot.data['contact']
    }
}
```

<small>
    More see: [https://blog.thoughtram.io/angular/2016/10/10/resolving-route-data-in-angular-2.html](https://blog.thoughtram.io/angular/2016/10/10/resolving-route-data-in-angular-2.html)
</small>
----

---

## Resources

- [Book: Angular Router](https://leanpub.com/router)