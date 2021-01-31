---
title: How to Present List Data in Angular?
date: 2020-07-04
tags: web development, Angular, recursion, RxJS, angular-tree-component
index_img: /images/thumbnail/list.jpg
---
**List** is one of the most common forms of presentating data in web application. One thing that makes list different, or harder to maanage than other UI components is that list can have recusive tree structure, and presenting recusive views can be hassle for developers. Consider the following list data in array:
```javascript 
var menu = [
    {
        label: 'item_1',
        id: '23452',
        hide: false,
        submenu: [
            {
                label: 'child_1',
                id: '62345',
                hide: false,
                submenu: [
                    {
                        label: 'grandchild_1',
                        id: '72433',
                        hide: false
                    },
                    ...
                ]
            },
            ...
        ]
    },
    ...
]
```
The list can go on, and you get the idea. 

### Intuition 
When comes to list in Angular, the first thought would be using the Angular native directive `*ngFor` to iterate through the items in the array. This is absolutely correct. However, when the list has one or more submenu attached to each or some of the items, using only *ngFor would not be the most efficient way. Consider the usage of *ngFor for the `menu` defined earlier:
```html 
<ul id="menu">
    <li *ngFor="let item of menu" [hidden]="item.hide">
        <span>{{item.id}}</span>
        <span>{{item.label}}</span>
        <ul *ngIf="item.submenu">
            <li *ngFor="let subitem of item.submenu">
                <span>{{item.id}}</span>
                <span>{{item.label}}</span>
                <ul *ngIf="subitem.submenu">
                    <li *ngFor="let subitem of subitem.submenu">
                        <span>{{item.id}}</span>
                        <span>{{item.label}}</span>
                    </li>
                </ul>
            </li>
        </ul>
    </li>
</ul>
```
The code above works for the example menu, but only for this example menu. What if the menu has more than three levels deep. Besides, the coding above violated `DRY coding principle`. So, what are the ways to present the lists like above gracefully? 

### A working solution: using external library
There are some external libraries providied for handling list in Angular, and `angular-tree-component` is one of them. angular-tree-component is a npm package that organizes list items into tree structure where user can expand, remove, add, and drag-and-drop data item within the tree. 

To use angular-tree-component, simply inject `<tree-root>` into html template. <tree-root> component normally accepts two attributes: `nodes` and `options`.
```html
<tree-root [nodes]="menu" [options]="options"></tree-root>
```
**menu** is the array that contains the list item. One thing handy about angular-tree-component is that if object contains nested object, angular-tree-component will identify and interpret those nested data as expandable submenu items of the list, which saves a lot of coding space and *ngIf conditions in html template.

**options** is an `ITreeOptions` object that overwrites angular-tree-component's default behaviours on how to parse the list data. 

See more implementations about angular-tree-component in [official documentation](https://angular2-tree.readme.io/).

As we can see that angualr-tree-component is like a black box that has encapsulated the process of iterating over the recursive tree structure of the list data, and developer doesn't need to write complicated html template that is hard to read. 

However, there is still limitation of using libraries to handle list in Angular. In some cases, the way of how the data gets displayed is constrainted by the framework of the library. For example in angular-tree-component, the sub-list can only be presented as an expandable since the template of the component is structured as the following:
```html 
<tree-root>
    <tree-node-collection>
        <div class="tree-node-level-1 tree-node tree-node-expanded tree-node-focused">
            <tree-node-expander></tree-node-expander>
            <tree-node-wrapper></tree-node-wrapper>
            <tree-node-children></tree-node-childre>
            <tree-node-drop-slot></tree-node-drop-slot>
        </div>
    </tree-node-collection>
</tree-root>
```
It means that the developers are given only limited freedom to decide how the data should be presented. What if we want the users to be able to navigate back and forth between main list and sub-list, and this scenario is certainly out of the scope of angular-tree-component. 

More importantly, developer should always be careful of injecting external thired-party dependencies into the main app. Every npm library has their lifecycle. If the library injected is a free open-source project that is outdated and not well maintained on a regular basis, depending on that package may introduce hidden issues to the main application. Therefore, Angular developers should have the mindset of trying to utilize Angular native packages at the first place before considering using external npm packages. 

### A better working solution: recursion
If we look closely to the menu list above, we can see that the template for each layer of the list data is the same, meaning that we can take this advantage and use recursion to map out the list. Suppose we have `menu-list.component.ts` and `menu-list.component.html`:

> menu-list.component.html
```html
<ul>
    <li *ngIf="isSubmenu" [ngClass]="{'hide': nextMenu}">
        <a (click)="collapse()">
            <span>Back</span>
            <i class="fa fa-arrow-left pull-right"></i>
        </a>
    </li>
    <li *ngFor="let item of data">
        <!-- If a list item -->
        <a *ngIf="!item.submenu" [ngClass]="{'hide: nextMenu'}">
            <span>{{item.label}}</span>
        </a>
        <!-- If a list folder -->
        <a *ngIf="item.submenu" [ngClass]="{'hide': nextMenu}" (click)="nextMenu = item.submenu;">
            <span>{{item.label}}</span>
            <i class="fa showarrow fa-angle-right pull-right"></i>
        </a>
    </li>
    <menu-list *ngIf="nextMenu" [data]="nextMenu" [isSubmenu]="true"></menu-list>
</ul>
```
> menu-list.component.ts 
```typescript 
import { Component, Input, OnInit } from '@angular/core';

@Component({
    selector: 'menu-list',
    templateUrl: './menu-list.component.html'
})

export class MenuListComponent implements OnInit {
    @Input() data: any;
    @Input() isSubmenu: boolean;
    nextMenu: any = null;

    constructor() {}
    ngOnInit(): void {}

    collapse(): void {
        this.data = null;
    }
}
```
In the code above, we inject `<menu-list>` component in the end of the template to present the next submenu. Now user is able to click on the list item which has a submenu, and navigate to that submenu. 

However, the code still has an issue when user tries to go back to the previous level from the submenu. Suppose the menu list have only two layers, that means the class object `MenuListComponent` will be initialized at least twice. On the first level, `nextMenu` is initialiiy null as defined in component. When user clicks on a folder item, `nextMenu` will be set to the submenu of that folder item, and gets passed to the next `MenuListComponent` class object. But when user navigates back to the first level, the value of `nextMenu` remains as the submenu of the clicked folder, and that makes all the items of the first level hidden. 

So how do we solve this? Specifically, how do we establish the communication between different `MenuListComponent` so the old class object can set the value of `nextMenu` to null when detects user clicking on the go-back button?

### RxJS library from Angular 
RxJS library adds on asunchronous programming paradigm to Angular, and allows components subscribe to events, timers, and promises which are broadcasted and propagated from other components. We can create a service called `EventService` that defines a `Observable` that pass message between multiple `MenuListComponent` class object. 

> event.service.ts 
```typescript 
import { Subject } from 'rxjs';

export class EventService {
    constructor() {}
    static menuListCollapse = new Subject();
}
```
> menu-list.component.ts 
```typescript 
import { Component, Input, OnInit } from '@angular/core';
import { EventService } from './event.service.ts';

@Component({
    selector: 'menu-list',
    templateUrl: './menu-list.component.html'
})

export class MenuListComponent implements OnInit {
    @Input() data: any;
    @Input() isSubmenu: boolean;
    nextMenu: any = null;

    constructor() {}
    ngOnInit(): void {
        EventService.menuListCollapse.subscribe(() => {
            this.nextMenu = null;
        });
    }

    collapse(): void {
        this.data = null;
        EventService.menuListCollapse.next();
    }
}
```
When user clicks on the go-back button, `collapse()` is invoked and broadcast the event of collapsing folder to all `MenuListComponent` that have been initialized. Hence the `MenuListComponent` initialized on the first level sets the value of `nextMenu` back to null after detecting the event of folder collapse.

I hope you found this article useful, and stay tuned for more of my own experiences and thoughts on web development!






