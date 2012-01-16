--- 
wordpress_id: 83
layout: post
title: Learnings on javascript scope chains, object prototype hierarchies, javascript "inheritance" and "constructors"
date: 2011-07-06 09:44:53 -04:00
wordpress_url: http://www.jessesanford.com/?p=83
---
I had to debug some very funny stuff going on in some of my models yesterday/this morning. It lead to some learnings about the internals of javascript inheritance and scope. Please excuse my run-ons and bad grammar!

First one of the interesting things about javascript is that variables can be declared locally inside a function and that function can be turned into an object using the "new" keyword but those local variables are not available to the object that the new keyword creates out of the function!

<em>Jesse = function() { var name = jesse };</em>

<em>jesse = new Jesse();</em>

<em>typeof </em><a href="http://jesse.name/"><em>jesse.name</em></a><em> === 'undefined'; //TRUE!</em>

&nbsp;

However if you use the this object within the context of the "constructor" it does work. The reason being that the "new" keyword creates a new object then inserts that object into the body of the "constructor" function's "scope chain" with the property name "this" that way any references to "this" within the constructor function reference the "new object" and not the this of any scope surrounding the constructor function or the call to the "new" keyword.

So if you were to think of the scope chain as an array like data structure where "global" akaÂ "window"Â scope is normally the last element in the array aka = array[array.length]

AND if you were to imagine the behavior of the javascript interpreter trying to de-reference a property as a walk through the array from element 0 to element array.length looking at every property on every object in each "cell" of the array and checking if their property names match the one in question.

Then the new keyword takes a blank object assigns the property name this to it and sticks it at the front of the aforementioned scope chain array and then evaluates the contents of the constructor using it.

Functions like call/apply/bind and the keyword "with" can be used to manipulate the scope chain programatically. However call/apply/bind can only manipulate the "this" property name and the "with" keyword comes with many caveats and performance hits.

In my opinion call/apply/bind are probably all shortcuts to simply creating a closure around a function with a pointer to the current "this".

Here is an example:

<em>var Jesse = function(){};</em>

<em>var jesse = new Jesse();Â //Look ma we now have an object with a reference to itself called "this"</em>

<em>jesse.firstName = "JESSE";Â //aka </em><a href="http://Jesse.this.name/"><em>Jesse.this.name</em></a><em> =Â "JESSE"; but not really because "this" is not available to Jesse only to jesse thanks to the "new" keyword</em>

<em>jesse.lastName = "SANFORD";</em>

<em>jesse.getFirstName = function() { return this.firstName; }; //yup you guessed it "this" is pointing at jesse</em>

<em>jesse.getLastName = function() { return this.lastName; };</em>

<em>//now here comes the fun!</em>

<em>var tom = { name: "TOM" }; //look ma another object! except this is an "object literal" who cares you say?... I agree I don't care... yet... see below.</em>

<em>jesse.getFirstName.call(tom); //yup this will return "TOM"</em>

<em>jesse.getFirstName.bind(tom)(); //same as before! //note the extra parens at the end are simply calling the function that is returned by the bind object which is simply the jesse.getFirstName function with the scope chain modified so that the reference to "this" is replaced with tom</em>

&nbsp;

Note that all of the above examples are very trivial and they are in the imperative style so they may not seem that handy. But they can also be used when doing asynchronous callback passing functional style

<em>//lets create a new function called printName which takes in two callback getter functions and print's their return values;</em>

<em>jesse.printName = function(fcb,lcb) { console.log(fcb()+" "+lcb()) };</em>

<em>//now you might expect the following to work:</em>

<em>jesse.printName(jesse.getFirstName, jesse.getLastName); //undefined undefined</em>

<em>//but it doesn't because now "this" points to the GLOBAL scope!Â </em>

<em>//check it out:</em>

<em>firstName = "global"; //note the lack of the use of var (pushing the variable into global scope)</em>

<em>Jesse = function(){};</em>

<em>Jesse.firstName = "parentobj";</em>

<em>Jesse.prototype.firstName = "prototype";</em>

<em>jesse = new Jesse();</em>

<em>jesse.firstName = "instance";</em>

<em>jesse.print(cb) { console.log(cb()) }; //just print whatever the callback returns</em>

<em>jesse.getFirstName = function() { return this.firstName }; //supposed to return the instance's foo variable</em>

<em>jesse.print(jesse.getFirstName); // 'global' !!! WHAT? :)</em>

&nbsp;

This may not be news to you but scope and functional programming can get wonky! So this is where the call and bind functions from above come in handy!

If we were to call the jesse.printName(fcb, lcb) function above with them we would get what we expected rather than "undefined undefined"

<em>jesse.printName(jesse.getFirstName.bind(jesse), jesse.getLastName.bind(jesse)); //"JESSE SANFORD"</em>

&nbsp;

OR if we re-wrote the printName function we could do the same with call (daisy chaining the passing of "this")

<em>jesse.printName =Â Â function(fcb,lcb) { console.log(fcb.call(this)+" "+lcb.call(this)) };</em>

<em>jesse.printName.call(jesse, jesse.getFirstName, jesse.getLastName);Â //"JESSE SANFORD"</em>

&nbsp;

But bind/call/apply may not look as clean as you might like. Also manipulating the scope chain just feels a little wrong to me. SO in some situations it makes more sense to use closures!

let's re-define our getFirstName and getLastName functions to remove the "this" from them. Instead let's have them reference a synonym called "self"

<em>jesse.getFirstName = function() { return self.firstName; };Â </em>

<em>jesse.getLastName = function() { return self.lastName; };</em>

&nbsp;

but what is self you say?Â it's whatever you make it!

check this out...

<em>var self = jesse;</em>

<em>jesse.printName(jesse.getFirstName,jesse.getLastName); //"JESSE SANFORD" !!</em>

Yes because now the interpreter when running through the blocks for getFirstName and getLastName will search the scope chain for a variable called "self" and the first one it finds will be used within them.

Again this example is very trivial and may not seem very useful. But think about it in reference to objects and their constructors. Remember a constructor when called ALWAYS has "this" pointed at it's new blank object instance. SO...

&nbsp;

Let's re-factor the Jesse parent object like so:

//We can also use the inheritance to define the Jesse parent obj more robustly for re-use... Hell let's just rename it to Person as you might have expected.

<em>var Person = function() {</em>

<em>Â  var self = this;Â //now self will point at WHATEVER new instance object is constructed by the "new" keyword when this "constructor" is called.</em>

<em>Â Â self.getFirstName = function() { return self.firstName; };Â </em>

<em>Â Â self.getLastName = function() { return self.lastName; };</em>

<em>Â Â self.printName = function(fcb,lcb) { console.log(fcb()+" "+lcb()) };</em>

<em>};</em>

<em>//now we can simply instantiate a new Person as jesse, name it and print the name!</em>

<em>jesse = new Person();</em>

<em>jesse.firstName = "JESSE";</em>

<em>jesse.lastName = "SANFORD";</em>

<em>jesse.printName(jesse.getFirstName, jesse.getLastName); //"JESSE SANFORD"!!!</em>

&nbsp;

NOTE: You might expect the following to work: (If you are confused as to why not... see the first example all the way at the top!)

<em>var Person = function() {</em>

<em>Â Â var self = this;</em>

<em>}</em>

<em>Person.prototype.getFirstName = function() { return self.firstName; };Â </em>

<em>Person.prototype.getLastName = function() { return self.lastName; };</em>

<em>Person.prototype.printName = function(fcb,lcb) { console.log(fcb()+" "+lcb()) };</em>

<em>jesse = new Person();</em>

jesse.firstName = "JESSE";

jesse.lastName = "SANFORD";

jesse.printName(jesse.getFirstName, jesse.getLastName); //ReferenceError: self is not defined (Because self is only available to the instance... NOT to the prototype which is where getFirstName and getLastName are defined)

&nbsp;

THIS ALSO WILL NOT WORKÂ ( again If you are confused as to why not... see the first example all the way at the top!)

<em>var Person = function() {Â </em>

<em>Â Â var self = this;Â </em>

<em>};</em>

<em>//if we construct a new jesse</em>

<em>jesse = new Person();</em>

<em>jesse.firstName = "JESSE";</em>

<em>jesse.lastName = "SANFORD";</em>

<em>//and define the following as above:</em>

<em>jesse.getFirstName = function() { return self.firstName; };Â </em>

<em>jesse.getLastName = function() { return self.lastName; };</em>

<em>jesse.printName = function(fcb,lcb) { console.log(fcb()+" "+lcb()) };</em>

<em>//we get as expected:</em>

<em>jesse.printName(jesse.getFirstName, jesse.getLastName); //ReferenceError:Â self is not defined (as above)</em>

&nbsp;

&nbsp;

A quick (albeit dangerous) fix to the above examples would be to use the "with" keyword and a local variable called self.

<em>var whoami = {self: jesse};</em>

<em>with(whoami) {</em>

<em>Â Â jesse.printName = function(fcb,lcb) { console.log(fcb()+" "+lcb()) };Â //"JESSE SANFORD"!!!</em>

<em>}</em>

However "with" is dangerous and slow and almost everyone regards it as something you should never use. The problem is what happens if you don't define "whoami" correctly RIGHT before you call "with" keyword. Will you remember WHERE you defined it? Will you have even been the one who defined it? "with" will cause the interpreter to walk the scope chain looking for properties on variables that match the variable names referenced within the block following it's use. This can cause variables from all different scopes to get used and as such very unpredictable output! It get's even worse if you redefine variables within the with block. You may try to set a value of a property within "whoami" and very well be creating new GLOBAL variables if you are not careful to make sure all variable names used are already contained within "whoami" as properties. See:Â <a href="http://stackoverflow.com/questions/61552/are-there-legitimate-uses-for-javascripts-with-statement/61676#61676">http://stackoverflow.com/questions/61552/are-there-legitimate-uses-for-javascripts-with-statement/61676#61676</a>

&nbsp;

&nbsp;

A safer alternative is in the works for javascript 1.7 using the "let" keyword

<em>let(self = jesse) {</em>

<em>Â Â jesse.printName = function(fcb,lcb) { console.log(fcb()+" "+lcb()) };Â //"JESSE SANFORD"!!!</em>

<em>}</em>

Of course there are not many interpreters for 1.7 yet though. (The above does not work in the current version of node.)

&nbsp;

A couple of other interesting notes creating objects/constructors and on prototype chains and when you can access them. *(see note above about object literals)

Basically if you create an object literal... you will never get access to it's prototype chain UNLESS you modify the "Object" object's prototype. This is the super-prototype in javascript. The only thing higher up in the order than Object is Object.prototype and then there is null. SO don't expect this to work:

<em>var tom = { firstName: "tom", lastName:"pytleski" };</em>

<em>tom.prototype.getLastName = function() { return this.lastName; }; // returnsÂ TypeError: Cannot set property 'getLastName' of undefined (because tom has no prototype! it's an object literal... yeah now we care :)</em>

<em>//However we can do this:</em>

<em>Object.prototype.getFirstName = function() { return this.firstName; };</em>

<em>tom.firstName = "tom";Â </em>

<em>tom.getFirstName(); //'tom'</em>

&nbsp;

But that is BAD! because you just added a getFirstName function to every object in your virtual machine.

<em>var one = new Number();</em>

<em>one.firstName = "two"; // funny huh :)</em>

<em>one.getFirstName(); //'two' // great now numbers have a getFirstName function!</em>

&nbsp;

So the correct way is definitely to create a constructor first. Even a blank constructor works:

<em>var Tom = function(){};Â </em>

<em>tom = new Tom();</em>

<em>tom.firstName = "tom"; //'tom'</em>

&nbsp;

BUT remember that the instance does NOT have a prototype chain. Trying the following will not work:

<em>tom.prototype.getFirstName = function() { return this.firstName; }; //TypeError: Cannot set property 'geFirstName' of undefined (because instances of objects don't have prototypes!)</em>

&nbsp;

However as you have seen above this does work:

<em>Tom.prototype.getFirstName = function() { return this.firstName; };</em>

<em>//and now:</em>

<em>tom.getFirstName(); // 'tom' //yes as expected.</em>

&nbsp;

Finally it may help to setup instance variables within the constructor right off the bat.

<em>var Tom = function() { this.firstName = "jesse"; this.lastName = "sanford"; };</em>

<em>Tom.prototype.getFirstName = function() { return this.firstName };</em>

<em>tom = new Tom();</em>

<em>tom.firstName = "tom"; //you can set their values just as you might expect</em>

<em>tom.getFirstName(); //'tom' //yup!</em>

&nbsp;

A note on inheritance... Everyone has probably figured this out already but I am repeating it here at the end if it's still not quite clear. The prototype heirarchy provides the single parent inheritance you have seen.

&nbsp;

<em>Person = function(){};</em>

<em>jesse = new Person();</em>

<em>tom = new Person();</em>

<em>Person.prototype.getFirstName = function() { return this.firstName; };</em>

<em>jesse.firstName = "jesse";</em>

<em>jesse.getFirstName(); //'jesse'</em>

<em>tom.getFirstName(); //'jesse' yup!</em>

<em>tom.lastName = "pytleski";</em>

<em>Person.prototype.getLastName = function() { return this.lastName; };</em>

<em>Person.prototype.lastName = "sanford";</em>

<em>tom.getLastName(); //'pytleski' yup! //Also note that the hierarchy is respected when the scope chain is inspected by the interpreter as expected</em>

<em>jesse.getLastName(); //'sanford'</em>

&nbsp;

Another quick tidbit about inheritance:

<em>//Guess what jesse.foo will be :)</em>

<em>Jesse = function(){};</em>

<em>Jesse.foo = "parentobj";</em>

<em>Jesse.prototype.foo = "prototype";</em>

<em>jesse = new Jesse();</em>

<em>jesse.foo; //? (scroll down)</em>

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

<em>//yeah it'sÂ 'prototype'!</em>

&nbsp;

Here were some of the good articles that lead to my solution:

<a href="http://www.academa.si/?content=http://www.academa.si/html/articles/js/professionalJavaScript/prototipesAndScopeChains.htm">http://www.academa.si/?content=http://www.academa.si/html/articles/js/professionalJavaScript/prototipesAndScopeChains.htm</a>

<a href="http://stackoverflow.com/questions/61552/are-there-legitimate-uses-for-javascripts-with-statement">http://stackoverflow.com/questions/61552/are-there-legitimate-uses-for-javascripts-with-statement</a>

<a href="https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/call">https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/call</a>

<a href="https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/function/apply">https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/function/apply</a>

more interesting stuff:

<a href="https://developer.mozilla.org/en/JavaScript/Reference/Functions_and_function_scope/arguments">https://developer.mozilla.org/en/JavaScript/Reference/Functions_and_function_scope/arguments</a>

<a href="http://unspecified.wordpress.com/2011/06/05/simulating-classes-with-prototypes-in-javascript/">http://unspecified.wordpress.com/2011/06/05/simulating-classes-with-prototypes-in-javascript/</a>
