---
title: Part 2 - A Look into Angular from a High-Level Perspective
date: 2020-08-24
tags: web development, Angular, architectural design
index_img: /images/thumbnail/angular.png
---
In the last part, we have looked into the matter of why having high scailibility of web app will benefit throughout the development process, and what are the solutions that help address the scailibility issue. By taking a brief insight of TypeScript, we have known the role of TypeScript in frontend development and how TypeScript incorporates with some of mainstream frontend frameworks, especailly Angular 2+. In this part, we'll take a look at the architecture of Angular from a high-level perspective, and how this specific architectural design guide and enforce developers to develop web app in a managable way.

### A way to understand a frontend framework 
To understand the philosophy of a frontend framework, a good starting point would be getting senses of what problems the authors of the framework were trying to solve. 

For example, Vue.js is most well-known as a `reactive` JavaScript framework. Before the very first commit of Vue.js, the author of Vue.js, Evan You was working at Google. The most of projects Evan was working on focused on the interactivity between human gesture and UI elements. To make the user interactions smooth and precise, the view components need to be highly synchronized with JavaScript data. 

Back then during those projects at Google, Evan was mostly working with AngularJS and Backbone.js. However these framework couldn't quite suit the needs as they are more emphasizing on building an application in a structural manners and confining the way how the frontend developers write their codes. Hence Evan started having intention of making a framework on his own that can fit well into the his works, a lightweighted framework that synchronize `JavaScript Object` and `DOM` in high speed. That's why in the early stage of Vue.js, we saw `two-way data binding` as one of the core features of Vue.js really took on the role of managing the user interactions in the application. And later on, more and more pieces started with this "data-binding" concept, fell into the ecosystem, and completed the Vue.js we see today.

> Note that for convenience, Angualar 2+ will be stated as Angular in below

Back from the tagent line, the point is, no matter what frontend framework it is, the framework must has the intention of solving certain domain of problems, around which every core feature was built. Same for AngularJS. AngularJS is initially designed to integrate multiple projects into one application, and make application more matainable and expandable. In 2016, the final version of Angular, a totally rewrite of AngularJS was released, aiming for pushing the limits of how application modules can be managed with the least efforts. 

To make the application parts communicate better with each other, Angular decomposes application into three layers: `presentation`, `abstraction`, and `core`. The core idea is to place the proper responsibilities into the proper layer, and each layer should only be care about a certain aspect of application.

### Presentation layer 
The responsibilities of presentation layer are presenting views to users and delegating the user actions. The basic unit in presentation layer is `Component`. A component is UI-related, which means anything a user sees in application, a dropdown menu, a navigation bar, the content display of any form, all come from component directly. A component contains three parts: `html template`, `style sheet`, and `view model`. For example, a component named `user-settings` will have a file structure as below:
```text
----user-settings
    |----user-settings.component.css
    |----user-settings.component.html
    |----user-settings.component.ts
```
`user-settings.component.ts` is the view model of user-settings component, which stands for the **VM** in **MVVM** architectural pattern. What a view model does is to bind data to the DOM, and handle the angular events(i.e., user actions). To put it simple, handling the UI logics. The following is what a typical view model looks like.
> user-menu-settings.component.ts
```typescript 
import { Menu, UserItem, User } from '@app/model';

var self: UserMenuSettingsComponent;

export class UserMenuSettingsComponent {
    @component({
        selector: 'app-usermenusettings',
        templateUrl: './user-menu-settings.component.html',
        styleUrls: ['./user-menu-settings.component.css']
    })
    dataListLabel: string;
    userMenu: Menu;
    userItemList: UserItem[],
    convertedUserItemList: any[];

    constructor(private translate: TranslateServie, private converter: DataConvertService) {
        self = this;
        self.dataListLabel = self.translate.instant(self.userMenu.data.label);
        self.getUserItemList();
    }

    getUserItemList(): void {   // Fetch userItemList
        User.isActiveStatus((isActive: boolean, uid: string) => {
            if(isActive) {
                self.userMenu = new Menu(uid);
                self.userMenu.getUserMenu(uid, (menu: UserItem[]): void => {
                    self.userItemList = menu;
                    self.convertUserItemList();
                })
            }
        })
    }

    convertUserItemList(): any[] {   // Convert userItemList to presentable list format
        return self.converter.processUserList(self.userItemList);
    }

    deleteUserItem(selectedUserItem: any) {
        self.userMenu.removeUserItem(selectedUserItem);
    }
}
```
In above code fragment, `Menu`, `UserItem`, and `User` are models from the abstraction layer, which provides component with user data and delegate user action(e.g: deleteUserItem()). `TranslateService` and `DataConvertService` are both injectable services. And they provides methods which process UI data. 

The whole idea behind view model is that the presentation layer should only care about how to present the fetched data in UI, and how to handle angular events. Regarding the actual implementations of business logics and fetching user data, those are hidden from presentation layer and presented as "black-boxes" to the components.

### Abstraction layer 
Abstraction layer is the layer underneath the presentation layer. The main role of abstraction layer is to be a bridge between UI and the "raw" data fetched from the server-side. The abstraction layer also takes in raw data and depends on the core services from the core layer. On the other hand, the abstraction also provides interfaces and states for presentation layer to access, hence, the components are able to retrieve user data and object states without knowing and caring how the things are actually done. The components should just call the methods from the abstration layer, and that's it.

The following is what a model in the abstraction layer looks like, in the same context of `UserMenuSettingsComponent` in the code snippet above.
> menu.model.ts
```typescript 
import { APIServicesConnector } from '@app/core';

export class Menu {
    userid: string;
    constructor(uid: string) {
        self.userid = uid;
    }

    getUserMenu(uid: string, callback: Function): void {
        APIServicesConnector.call({
            COMMAND: 'GETUSERMENU',
            USERID: self.userid
        }, callback);
    }
}
```
The model above is `Menu` which checks if the user status is active. Instead of exposing to the built-in `http service` of Angular and retrieving data from the server-side, the model comsumes data through the core layer, that is, `APIConnector` in this case. The models or the services in the abstraction layer should only focus on abstracting the way of how the presentation layer connects to the core services and the data.

### Core layer 
The main responsibility of the core layer is to communicate with the outside world, that is, the networking layer and the server-side. Core layer is where all the models and components are built on. Consider the core layer being the central kitchen in a resturant(an application), the cooks take the ingredients(the raw data from server-side), cook them following the recipes(manipulate data based on the business logics), and the dishes are to be delivered by the waiters(the model and services) and presented to the customers. 
> api-services-connector.service.ts
```typescript 
import { Injectable } from '@angular/core';
import { HttpClient, HttpShrBackend } from '@angular/common/http';

var self: APIServicesConnector;
@Injectable({
    provideIn: 'root'
})

export class APIServicesConnector {
    http: HttpClient;
    requestHeader: any = {
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json; charset=utf-8'
        }
    }

    constructor() {
        self = this;
        self.http = new HttpClient(new HttpXhrBackend({ build: () => new XMLHttpRequest() }));
    }
    
    static call(apidata: any, callback: Function): void {
        var url = '/app/examples/user';
        self.http.post<any>(url, apidata, {...self.requestHeader}).subscribe(response => {
            if(response.STATUSCODE === '200') {
                callback(response.data.userlist);
            }
        });
    }
}
```
The code above is `APIServicesConnector` service in the core layer, and this is where the http request is created from the client-side and sent to the backend. APIServicesConnector is the service that interacts with the native Angular http module, similar to all other core services that hide outside world communications from the abstraction and presentation layers. 

Other than the http service, the services that have dependencies to other native Angular services such as `RxJS events`, `Router`, and `Translation` should also live in the core layer. Normally these services have the global scope to the application, and can be accessed by any model, service, and component from the upper layers. 

### Upcoming in the next
Structuring the app modules properly into the layers doesn't only make the codes more readable from developer's perspective, but also benefits scaling the application. For example, a new component will always go to the presentation layer, and reuse the models that are already in the abstraction layers. 

In the next, we will look at some other aspects of Angualar including unidirectional data flow and state management. Hope you find this article helpful, and stay tuned for more of my own experiences and thoughts about the modern web development!

















