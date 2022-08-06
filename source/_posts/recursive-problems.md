---
title: Working with Recursive Problems in Frontend
date: 2020-09-20
tags: web development, recursion
index_img: /images/thumbnail/recursion.png
---
Recursion is a standard program procedure used to solve the problems recursively. We heard about recursion a lot, and it can exist in any place that you can think of. Let's take a look at one of the recursive problems which may usually be seen in frontend development. 

Many web apps may have components that display a list of data recursively, most likely, a tree structure of data. For example, Google Earth. We all know that initially google earth shows all the continents, and as you continuously zoom in, more and more detailed geographic information starting from countries to a specific district area get displayed. From a perspective of developer, the most intuitive way to implement this is using recursion, and the data on the client side may look like this:
```javascript
[
    {
        name: 'North Amercian',
        type: 'CONTINENT',
        childs: [{
                name: 'Canada',
                type: 'COUNTRY',
                childs: [{
                    name: 'British Columbia',
                    type: 'PROVINCE',
                    childs: [...// cities
                    ]
                },
                ...// Other Canadian provinces
                ]
            }]
    },
    {
        name: 'Asia',
        type: 'CONTINENT',
        childs: [...// Asian countries
        ]
    },
    ...// Other continents
]
```
Of course, the program procedures and the algorithms used are much more advanced and complicated in real Google Earth app. But let's just boil it down to a simplifed version and see how to incorporate such kind of data into the app interface.

Sometimes, the nested object data is not the best data structure to store in the backend because of the memory size of Javascript object and also the specific way how relational database store the records. More likely, the data may be stored as follow:
```javascript 
[
    {id: '3920', parent: '7452'},
    {id: '9683', parent: '1046'},
    {id: '3892', parent: '2950'},
    {id: '0372', parent: '7452'},
    ...
]
```
Most of list-related UI packages including `angular-tree-component`, `angular-material-tree` accepts list data only in form of nested JavaScript object. Therefore, we need to recursively map those item relations into nested object. 

Before getting into the recursion, we can process the array into a cleaner data strucutre so the relations between each item can be properly laid out. Here we can use `Hash Map` to store the relations in key-value pairs, where the key is the id and the value is an array containing all the child items of that id, something like below:
```text
key             value 
-------------------------
'7382'          ['0471', '4829', '1042']
'5728'          ['8492']
'0362'          ['3948', '9402', '1482', '4999']
```
Note that if a key doesn't appear in the map, that means, that id is a leaf node which has no child. The following is the implementation of mapping relation array into a JavaScript hashmap.
```typescript
relations: Map;
buildMap(): void {
    this.array.forEach((element: any) => {
        let parent = element.parent;
        // If this parent id already stored in the hashmap
        if(this.relations.has(parent)) {
            let childs = this.relations.get(parent);
            this.relations.set(parent, [...children, element.id]);
        }
        else {
            this.relations.set(parent, [element.id]);
        }
    })
}
```
Given the map and the root node, now we are able to recursively construct the tree based on the relations. 
```typescript 
interface nodeStruct {
    id: string,
    childs?: nodeStruct[]
}
buildTree(): void {
    var self = this;
    var obj: nodeStruct;
    var buildNode = function(_id: string): nodeStruct {
        if(!self.relations.has(_id)) {  // Base case
            return {id: _id}
        }
        var node: any = {};
        var childs: nodeStruct[] = [];
        var _childs = self.relations.get(id);
        for(let i = 0; i < _childs.length; i++) {
            childs.push(buildNode(_childs[i]));
        }
        node.id = _id;
        node.childs = _childs;
        return node;
    }

    // Initialize tree with root node
    obj = buildNode(self.root.id);
    return obj;
}
```
Here by calling `buildTree()` we can get a nested object data based on all the relations. Although most of times the straight-up recursion can solve the recursive problems, however, the implementation can get very expensive in terms of the used memeory space. Since each recursion adds on another function call layer to the stack, without choosing boundary case carefully, the stack can easily get fully occupied and run into `maximum stack exceeds` issue. Furthermore, the time complexity of recursion could have the worst case of O(n^n) since it's traversing each node of a tree. 

Therefore, it is always a better mindset to think of iterative solutions to a recursive problem first. For example, many dynamic problems can go with bottom-up approach where starts with the simplest cases, and then iteratively build up the array that contains the solution for each case, until reaches to the target output.

I hope you find this article useful. Stay tuned for more of my sharings about the modern web development!

