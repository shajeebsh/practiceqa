---
title: "Jquery advncd"
date: 2012-07-25 06:41:13 
author: shajeebhameed
layout: page
slug: jquery-advncd
status: publish
---

<http://www.tuttoaster.com/some-good-and-advanced-jquery-techniques/> jQuery Functions and Techniques to Keep in Mind Nowadays jQuery is used in so many sites that we can’t even count them. There is a reason, jQuery is probably the most famous JavaScript framework. As developers we have to think beyond. We should be able to use the most advanced techniques of every language or framework we want to know. Here I’ll explain to you some of the good practices and techniques we should be using right now for taking advantage of all this framework can offer us. Advanced Selectors In jQuery we can use a lot of selectors, making them very accurate. Let’s see step by step some of them and try to get the point to be able to use them in other contexts. Selectors based on attributes Almost every element in HTML can have attributes, for instance: 1  2 In only two html elements we have 9 attributes. With jQuery we can select the elements knowing their attributes and values. Take a look at this selectors: You can see many others very similar to these ones, in the official documentation. But the point is the same, attributes and theirs values. Multiple selectors If you are working on an extensive site, the selectors can become hard. Sometimes it’s better just split them into two or three selectors to make the code easier. Actually this is a very simple and basic thing but it’s truly useful, and we have always to keep it in mind. 01 $(document).ready(function(){ 02 //All the images whose width is 600px OR height is 400px 03 $("img[width=600],img[height=400]").click(function(){ 04 alert("Selected an image whose width is 600px OR height is 400px"); 05 }); 06 07 //All p elements with class orange_text, divs and images. 08 $("p.orange_text, div, img").hover(function(){ 09 alert("Selected a p element with class orange_text, a div OR an image."); 10 }); 11 12 //We can also combine the attributes selectors 13 //All the jpg images with an alt attribute (the alt's value doesn't matter) 14 $("img[alt][src$='.jpg']").click(function(){ 15 alert("You selected a jpg image with the alt attribute."); 16 }); 17 }); Widget selectors As well as the usual selectors there are a few ones based on some attributes or positions of elements.Let’s see the more important ones: 01 $(document).ready(function(){ 02 //All the hidden images are shown 03 $("img:hidden").show(); 04 05 //The first p is going to be orange 06 $("p:first").css("color","orange"); 07 08 //Input with type password 09 //this is $("input[type='password']") 10 $("input:password").focus(function(){ 11 alert("This is a password!"); 12 }); 13 14 //Divs with paragraph 15 $("div:has(p)").css("color","green"); 16 17 //We can also combine them. with () 18 //All not disabled checkboxes 19 $("input:checkbox(:not(:disabled))").hover(function(){ 20 alert("This checkbox is working."); 21 }); 22 23 }); As you can see we have a wide spectrum of selectors, and they aren’t independent so we can combine them (like the last example). Understanding the Structure of a Site I know it could seem complicated but actually it’s not at all and we have to do it if we want to use some selectors and, therefore, methods that work on them. We can see a site as a reversed tree, with the root at the top and the rest of elements coming from there. Take a look at this code and try to imagine a tree, whose root is the body tag. 01 ... 02 

03 

04 

# Create an Account!

05 06 Personal Information 0708 09 10 11 12 Message 

13 

14  15 

1617 Footer message 

18  Let’s see this code as a tree: Easy, right? So from now on we have to see the (x)html as a tree, but why all this theory about botanic if we want to be developers? The answer is in the next point. Selecting and Transforming the Tree Selecting Just like in a tree there are parents and children. In jQuery we can take advantage of this structure. Imagine we have the same html, but now we want to select the p inside the div with id “main”, but we don’t want to change anything about the p element (in fact, if we were going to change it just for the jQuery code were a bad practice.) Here we’ve got 3 possible solutions: 1 $("#wrapper").children('#main').children('p').css("color","orange"); 2 3 $("#wrapper").children().children('p').css("color","orange"); 4 5 $("#main").children('p').css("color","orange"); The children method allows you to select an element which is under of other element in the tree. If we pass it a selector, the method will select only the elements we want, but if we don’t all the children of a parent will be selected. Take a look at the difference between the first and the second selector. The only difference is we specify nothing in the second one so every single children will be selected. The point here is if we see the sketch, we don’t have any other children under the div wrapper, that’s the reason we are getting the same effect. Adding elements Now we are going to add an element to our tree. This element could be a paragraph, a div, a form or any other, let’s imagine we want to add a list, something like: 1 

2 
    * Dog

3 
    * Cat

4 
    * Frog

5 That’s just a string, so we can have it in that way in our JavaScript code: 01 var list = " 02  \n" 
  03 
  \+ " 
  04 
    * Dog

05 \n" 06 \+ " 07 
    * Cat

08 \n" 09 \+ " 10 
    * Frog

11 \n" 12 " 13 "; Now we have to add that string to the html somewhere. For instance. after the p element we have selected before. And finally we can type the whole code: 01 $(document).ready(function(){ 02 var list = " 03  \n" 
  04 
  \+ " 
  05 
    * Dog

06 \n" 07 \+ " 08 
    * Cat

09 \n" 10 \+ " 11 
    * Frog

12 \n" 13 " 14 "; 15 16 $("#wrapper").children('#main').append(list); 17 }); We are literally appending the string to the in the HTML, the result is that the list is shown after the p element. Removing elements This is not difficult at all, but it has to be here if we are talking about transforming the tree. Let’s remove the famous paragraph which we selected before (notice we can reuse the selector). 1 $("#wrapper").children('#main').children('p').remove(); Be careful because with this code all the elements inside the one we select will be removed. If we have a div with a list inside and we remove the div the consequence is both div and list will be removed. Looking under the hood jQuery is like an iceberg, there is a lot of things under the surface, some of them are shown in this section. Bind Bind is a method which allows us to attach an event (click, hover, focus…) to a method. It’s like saying: “When the user clicks here then call this method”. It sounds familiar, doesn’t it? That’s because we are using constantly. Take a look at this example: 1 $(document).ready(function(){ 2 $("#id").click(function(){ 3 alert("That click was amazing!"); 4 }); 5 }); Well, that click() method, is a wrapper for this code: 1 $(document).ready(function(){ 2 $('#id').bind('click', function () { 3 alert("That click was amazing!"); 4 }); 5 }); We can also unbind an event from an element with the unbind() method. Defining your own jQuery method So far we have seen some methods, such as click, bind, hover….etc, but how can we make our own methods and call them with something like: $(‘selector’).mymethod()? The solution is below. Let’s make a method for alerting the value of an element: 1 //The name will be alertVal 2 jQuery.fn.alertVal = function() { 3 var element = $(this[0]); //That's our element 4 if (element.val()) 5 alert(element.val()); //That's our element's value 6 }; 7 8 //This is the way we can use it 9 $("selector").alertVal(); Callbacks are usually the solution Callbacks allow us to execute a method when other finishes. You can imagine the callback method says to the other: “When you finish call me, because I have to do my work.” But now the question is: how can we use them? 1 $(document).ready(function(){ 2 myCallBack = function(){ 3 alert("I'm a callback alert."); 4 } 5 6 //When the get finishes then myCallBack is executed 7 $.get('myhtmlpage.html', myCallBack); 8 9 }); If the function has arguments remember we have to do it in this way: 1 $(document).ready(function(){ 2 $.get('myhtmlpage.html',function(){ 3 myCallBack(param1, param2); 4 }); 5 }); Conclusion These are only some techniques for getting better knowledge about jQuery. Of course there are a lot of other techniques more advanced than these ones, but take this post as the first step to become a really good jQuery developer. By the way, jQuery has a great documentation so check the links out. Official Documentation Learning jQuery (a good blog to learn jQuery) Image by Shutterstock The Testking 650-393 online training is the best source to advanced jquery techniques. Download Testking 70-685 design tutorials with expert Testking 642-681 reviews to get unique design ideas.
