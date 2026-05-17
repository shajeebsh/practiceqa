---
title: "Javascript"
date: 2012-07-18 10:11:14 
author: shajeebhameed
layout: page
slug: javascript
status: draft
---

### Generic

  * Do you use any libraries? If so, which ones and why do you like them?
  * Have you written any libraries yourself, such as DOM helpers?
  * Do you use any server-side JavaScript frameworks?
  * What's the difference between [ECMAScript](<http://en.wikipedia.org/wiki/ECMAScript>) and JavaScript?
  * Do you use any JavaScript code validators?
  * Which JavaScript books have you read/recommend?
  * Do you unit test your JavaScript code?



### (Beginner/Medium) specific questions

  * Why does nearly every object have a `toString` method?
  * Do you know which interpreter Mozilla Firefox uses? Or any other of the browsers?
  * Does JavaScript support lambda functions?
  * What's the most useful JavaScript function you've created/use?
  * Is there block scope in JavaScript?
  * Can you explain how Ajax/XMLHttpRequest works?
  * Does JavaScript support classical inheritance?
  * Can you give me a code snippet that uses the `with` statement?
  * Do you know what Greasemonkey is? Have you used it?
  * Do you think innerHTML is evil?
  * What is JSON?



### Advanced questions

  * Can you give me an example of a generator?
  * How does [JSONP](<http://en.wikipedia.org/wiki/JSONP>) work?
  * An example of a singleton pattern?
  * What's the difference between undefined and undeclared?
  * Have you done any animation using [Raphaël](<http://en.wikipedia.org/wiki/Rapha%C3%ABl_%28JavaScript_library%29>) or the canvas element?
  * Are you familiar with Web Workers?
  * Do you do any profiling? What tools do you use?
  * Have you read the new ECMAScript specification? What new features are in there?



### People questions

  * Who initially wrote ECMAScript? Do you know where he works and his title?
  * What's the name of that guy who wrote jQuery?
  * Who wrote [JSLint](<http://en.wikipedia.org/wiki/JSLint>)?



### Cross-browser/DOM API questions

  * The standard `addEventListener` is supported in which browsers?
  * Which browsers incorrectly implemented getElementByID such that they also return elements whose name attribute is the id?



### Code snippet questions

**[1]:**
    
    
    <a href="#">text</a><br><a href="#">link</a>
    <script>
        var as = document.getElementsByTagName('a');
        for ( var i = as.length; i--; ) {
            as[i].onclick = function() {
                alert(i);
                return false;
            }
        }
    </script>
    

Why do the anchors, when clicked on, alert `-1` instead of their respective counter inside of the loop? How can you fix the code so that it does alert the right number? (Hint: closures) **[2]**
    
    
    function User(name) {
        this.name = name;
    };
    
    var j = User('Jack');
    alert(j.name)
    

Why would this code not work as expected? Is anything missing? **[3]:**
    
    
    Object.prototype.jack = {};
    
    var a = [1,2,3];
    
    for ( var number in a ) {
        alert( number )
    }
    

I'm iterating through this array I defined, it has 3 elements in it.. the numbers 1 2 and 3.. Why on earth is jack showing up? **[4]:**
    
    
    people = {
        1:'Jack',
        2:'Chloe',
        3:'Bruce',
    }
    
    for ( var person in people ) {
        alert( person )
    }
    

Why is this not working in Internet Explorer? **[5]**
    
    
    <script>
        (function() {
            var jack = 'Jack';
        })();
        alert(typeof jack)
    </script>
    

Why does it alert undefined when I declared the jack variable to be 'Jack'? **[6]:**
    
    
    <http://file.js>
    

Why isn't it alerting 2? **[7]:**
    
    
    array = [1,2]; alert( typeof array )
    

Why does it say `object` and not array? How would I detect if its an array? **[8]:**
    
    
    <a id="clickme">click me!</a>
    <script>
        var a = document.getElementById('clickme');
        a.onclick = function() {
            alert(this.innerHTML)
            setTimeout( function() {
                alert( this.innerHTML );
            }, 1000);
        };
    </script>
    

Why does the second alert say `undefined`? **[9]:**
    
    
    <p id="test">original</p>
    <script>
        var test = document.getElementById('test');
        test.innerHTML.replace('original', 'FOOBAR');
    </script>
    

How come it doesn't replace the text with FOOBAR?? **[10]:**
    
    
    function identity() {
        var name = 'Jack';
        alert(name);
        return
        name
    };
    var who = identity();
    alert(who)
    

Why does it first alert Jack, then undefined? **[11]:**
    
    
    var number = '08',
        parsed = parseInt(number);
    
    alert(parsed)
    

The alert should give me 8.. why doesn't it give me 8? **[12]:**
    
    
    <script>
        var xhr = new XMLHttpRequest();
        xhr.open("GET", "http://www.google.com", true);
        xhr.onreadystatechange = function(){
            if ( xhr.readyState == 4 ) {
                if ( xhr.status == 200 ) {
                    alert( xhr.responseText )
                } else {
                    document.body.innerHTML = "ERROR";
                }
            }
        };
        xhr.send(null);
    </script>
    

How come I can't retrieve Google's homepage text with [XHR](<http://en.wikipedia.org/wiki/XMLHttpRequest>)[6] from my localhost? **[13]:**
    
    
    <script>
        var ticket = true;
    
        if (!ticket)
            alert('you need a ticket');
            alert('please purchase a ticket.')
    </script>
    

`ticket` is set to true. Why is it alerting that I need to purchase one? **[14]:**
    
    
    <script>
        var blogEntry = 'Today I woke up
            to the smell of fresh coffee';
    
        alert(blogEntry)
    </script>
    

How come it doesn't alert with the blog entry? Everything seems right. **[15]:**
    
    
    alert( [typeof 'hi' === 'string', typeof new String('hi') === 'string' ]  )
    

I have two strings, but the second evaluation doesn't become true. Is this a potential bug? How come it's not true? 

### Best practices
    
    
    <a href="#" onclick="javascript:window.open('about.html');">about</a>
    

Would you change anything in the prior code example? Why? 
    
    
    <a href="site.html" onmouseover="changeImages('button1', 'images/button1over.png'); return true;" onmouseout="changeImages('button1', 'images/button1.png'); return true;">site</a>
