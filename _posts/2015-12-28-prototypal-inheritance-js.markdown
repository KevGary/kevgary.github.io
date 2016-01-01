---
layout: post
title:  "Prototypal Inheritance in JavaScript"
date:   2015-12-28 14:35:29 -0400
categories: tutorials
---
####Source: [https://github.com/KevGary/prototypal-inheritance-js-demo](https://github.com/KevGary/prototypal-inheritance-js-demo)

##Overview

##Getting Started
Follow along in the tutorial by creating a new directory with four javascript files:
{% highlight bash %}
$ mkdir prototypal-inheritance-js-demo
$ cd prototypal-inheritance-js-demo
$ touch television.js company.js dice.js high-roller.js
{% endhighlight %}

Or git clone final source code for reference and reuse:
{% highlight bash %}
$ git clone https://github.com/KevGary/prototypal-inheritance-js-demo
{% endhighlight %}

##Tutorial

###Object
This is an empty JavaScript [object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object): 
{% highlight javascript %}
var object = {};
{% endhighlight %}
A JavaScript object is either empty or has key-value pairs:
{% highlight javascript %}
var television = {
  screenSize: 65,
  resolution: '3840 x 2160',
  brand: {
    make: 'Panasonic',
    model: 'TX-65CZ952'
  },
  inventory: 20
};
{% endhighlight %}
{% highlight javascript %}
var alphabet = {
  0: 'A',
  1: 'B',
  2: 'C',
  3: 'D'
  //rest of letters
}
{% endhighlight %}
Access, add, modify, or delete key-value pairs from an object:
{% highlight javascript %}
//access
television[screenSize] //65;
//add
television[technology] = 'OLED';
//modify
alphabet[0] = 'Z';
//delete
delete alphabet[1];
{% endhighlight %}
The same is possible with dot notation for television but not for alphabet since the keys are integers:
{% highlight javascript %}
//access
television.brand.make //'Panasonic'
//add
television.HD = true;
//modify
television.inventory--;
//delete
delete television.resolution
{% endhighlight %}

###Constructor
Instead of hard-coding an object like above, use a constructor function to generate __new__ objects:
{% highlight javascript %}
function Television(screenSize, resolution, brand, inventory) {
  this.screenSize = screenSize || 32;
  this.resolution = resolution || '1360x768';
  this.brand = {make: arguments[2][0], model: arguments[2][1] } || {make: 'generic', model: 'generic'};
  this.inventory = inventory || 10;
};

var panasonic4K = new Television(65, '3840 x 2160', ['Panasonic', 'TX-65CZ952'], 20);
var sony75 = new Television(75, '3840 x 2160', ['Sony', 'KD-75X9405C']);
var defaultTV = new Television();
{% endhighlight %}

###Prototype

#####-television.js
JavaScript objects have a [prototype](https://developer.mozilla.org/en/docs/Web/JavaScript/Inheritance_and_the_prototype_chain). Paragraph two reads:

_When it comes to inheritance, JavaScript only has one construct: objects. Each object has an internal link to another object called its prototype. That prototype object has a prototype of its own, and so on until an object is reached with null as its prototype. null, by definition, has no prototype, and acts as the final link in this prototype chain._

If we wanted to add two methods, sellTV and buyTV, we could do the following:
{% highlight javascript %}
function Television(screenSize, resolution, brand, inventory) {
  this.screenSize = screenSize || 32;
  this.resolution = resolution || '1360x768';
  this.brand = {make: arguments[2][0], model: arguments[2][1] } || {make: 'generic', model: 'generic'};
  this.inventory = inventory || 10;
  this.buyTV = function() {
    this.buyTV = function(quantityBought) {
    quantityBought = quantityBought || 1;
    this.inventory += Number(quantityBought);
  };
  this.sellTV = function(quantitySold) {
    quantitySold = quantitySold || 1;
    if(this.inventory <= 0) {
      return 'Out of Stock';
    } else {
      this.inventory -= Number(quantitySold);  
    }
  };
};
{% endhighlight %}
Let's put these methods on the prototype of Television.  Every __new__ object that inherits from Television will also inherit from its prototype:
{% highlight javascript %}
function Television(screenSize, resolution, brand, inventory) {
  this.screenSize = screenSize || 32;
  this.resolution = resolution || '1360x768';
  this.brand = brand || {make: 'generic', model: 'generic'};
  this.inventory = inventory || 10;
};
Television.prototype.buyTV = function(quantityBought) {
  quantityBought = quantityBought || 1;
  this.inventory += Number(quantityBought);
};
Television.prototype.sellTV = function(quantitySold) {
  quantitySold = quantitySold || 1;
  if(this.inventory <= 0) {
    return 'Out of Stock';
  } else {
    this.inventory -= Number(quantitySold);  
  }
};
{% endhighlight %}
Refactor prototype methods for to keep things clean:
{% highlight javascript %}
Television.prototype = {
  buyTV: function(quantityBought) {
    quantityBought = quantityBought || 1;
    this.inventory += Number(quantityBought);
  },
  sellTV: function(quantitySold) {
    quantitySold = quantitySold || 1;
    if(this.inventory <= 0) {
      return 'Out of Stock';
    } else {
      this.inventory -= Number(quantitySold);  
    }
  }
};
{% endhighlight %}
Again, all __new__ objects will inherit from Television.prototype:
{% highlight javascript %}
var panasonic4K = new Television(65, '3840 x 2160', ['Panasonic', 'TX-65CZ952'], 20);
panasonic4K.buyTV(10);
panasonic4K.sellTV(5);
{% endhighlight %}

###Prototype chain

#####-company.js
As found in the github project's file company.js, I have provided a loaded example offering a small glimpse at the power of JavaScript's prototype chain and prototypal inheritance:
{% highlight javascript %}
//-----Prototype Chain-----
function Company(companyName) {
  this.companyName = companyName;
  this.products = [];
  this.employees = [];
}
Company.prototype = {
  newProduct: function(productName, price) {
    this.products.push({name: productName, price: price});
  },
  addEmployee: function(employeeName, position) {
    this.employees.push({name: employeeName, position: position})
  }
}

function Subsidiary(companyName, products, employees) {
  Company.call(this, companyName, products, employees);
}
Subsidiary.prototype = Company.prototype;
//add methods to Subsidiary prototype
Subsidiary.prototype.acquireSubsidiary = function(otherSubsidiary) {
  for(var i = 0; i < otherSubsidiary.employees.length; i++) {
    this.employees.push(otherSubsidiary.employees[i]);
  }
  for(var i = 0; i < otherSubsidiary.products.length; i++) {
    this.products.push(otherSubsidiary.products[i]);
  }
}

//compare prototypes
console.log(Company.prototype);
console.log(Subsidiary.prototype);

var myAmazingCompany = new Company('AmazingCompany');
var myAmazingSubsidiary = new Subsidiary('AmazingSubsidiary');
var myOtherAmazingSubsidiary = new Subsidiary('OtherAmazingSubsidiary');

myOtherAmazingSubsidiary.newProduct('Cooler-Can', 9.99);
myOtherAmazingSubsidiary.addEmployee('Michael', 'Manager');
myOtherAmazingSubsidiary.addEmployee('Sally', 'Accountant');

//watch the acquisition happen
console.log(myAmazingSubsidiary);
myAmazingSubsidiary.acquireSubsidiary(myOtherAmazingSubsidiary);
myOtherAmazingSubsidiary = null;
console.log(myAmazingSubsidiary);
console.log(myOtherAmazingSubsidiary);
{% endhighlight %}
There are two constructor functions, Company and Subsidiary. Subsidiary's prototype is initialized to be the same as Company's and then a method named acquireSubsidiary is added.  Multiple methods and/or properties can be added to the object's prototype in this way.  The rest of the code snippet displays the result through console logs.

### Modules- 

#####-dice.js, high-roller.js
A constructor function and its prototype can be made into a module.  Modules can be exported and used and reused in other parts of a codebase.  Provided below is ~20 lines of code that reiterates the lessons in this tutorial so far and can be used to play any dice game imaginable. In dice.js:
{% highlight javascript %}
function Dice(sides, numberOfDice, diceColor, numberColor) {
  this.sides = sides || 6;
  this.numberOfDice = numberOfDice || 1;
  this.diceColor = diceColor || 'white';
  this.numberColor = numberColor || 'black';
}

Dice.prototype.roll = function(numberOfRolls, numberOfDiceToRoll) {
  var resultsArray = [];
  this.numberOfDiceToRoll = numberOfDiceToRoll || this.numberOfDice;
  for(var i = 0; i < numberOfRolls; i++) {
    resultsArray.push([])
    for(var j = 0; j < this.numberOfDiceToRoll; j++) {
      resultsArray[i].push(Math.floor(Math.random() * this.sides) + 1);      
    }
  }
  return resultsArray;
}
{% endhighlight %} 
Export the Dice constructor function by including 1 line at the bottom of the code snippet:
{% highlight javascript %}
module.exports = Dice;
{% endhighlight %}
In dice-game-one.js, require the Dice constructor as a Dice instance and create two players, each with their own dice:
{% highlight javascript %}
var DiceInstance = require('./constructor.js');
var playerOne = new DiceInstance(6, 2);
var playerTwo = new DiceInstance(6, 2);
{% endhighlight %}
DiceInstance is now the same as Dice in dice.js and as so playerOne and playerTwo inherit everything (properties/methods and prototype) from the original Dice constructor function.  Below is a game called highRoller included with the entire high-roller.js file: 
{% highlight javascript %}
var DiceInstance = require('./dice.js');
var playerOne = new DiceInstance(6, 2);
var playerTwo = new DiceInstance(6, 2);

function HighRoller(numberOfPlayers, numberOfTurns) {
  this.numberOfPlayers = numberOfPlayers || 2;
}
HighRoller.prototype = {
  play: function () {
    if(arguments.length != this.numberOfPlayers) return 'All players must roll';

    var score = {}
    var highScore = 0;
    var winner;

    for(var i = 1; i <= arguments.length; i++) {
      score[i] = arguments[i-1][0][0] + arguments[i-1][0][1];
    }
    for (key in score) {
      if(score[key] > highScore) {
        highScore = score[key];
        winner = key;
      }
    }
    return [score, winner];
  }
}

var gameInstance = new HighRoller();
console.log(gameInstance.play(playerOne.roll(), playerTwo.roll()));
{% endhighlight %}
HighRoller is yet another constructor function and its prototype has the method play.  play takes in all players' rolls, sums them individually, and returns the round's score and winner determined by highest score.  The [Github project](https://github.com/KevGary/prototypal-inheritance-js-demo) shows HighRoller being played with three players.

The /dice.js file can be required in the same way in as many files as needed.   





 
