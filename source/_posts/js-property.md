---
title: The Difference Between __proto__ And prototype
date: 2017-02-21 16:03:37
categories: JavaScript #文章文类
tags: [Prototype] #文章标签，多于一项时用这种格式
---
![JavaScript proto chain](/blog/img/js_proto_chain.png)
There are several answers here how to check if a property exists in an object.

I was always using

if(myObj.hasOwnProperty('propName'))
but I wonder if there is any difference from
if('propName' in myObj){

<!--more-->

# An example

``` js
var test = function() {}

test.prototype.newProp = function() {}

var instance = new test();

instance.hasOwnProperty('newProp'); // false
'newProp' in instance // true

```

# another example
``` js
var o  = {"name":"sun nan","university":"Deakin","course":"Bachelor of Information Technology (Programming)-Deakin","email":"417757848@qq.com","first_name":"Nan","last_name":"Sun","date_of_birth":"1993-09-13"}

o.prototype.print = function(){
    for(var x in y){
        console.log(x, y[x]);
    }
}
```
undefined error will be raise



# next example

``` js
  //In world of JavaScript, very function can be used as a constructor function.
  //Let's make a Goddess Object
  function Goddess () {
    this.name = "Alice";
  }
  
  var hand = {
    whichOne: "right hand",
    someFunction: function(){
      console.log("not safe for work.");
    }
  };

  Goddess.prototype = hand; 

  //then, we can use Goddess as a constructor, construct an Object
  var myObject = new Goddess();
  console.log(myObject.__proto__ === Goddess.prototype) //true
```