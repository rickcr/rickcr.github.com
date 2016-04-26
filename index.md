---
title: learntechnology.net
layout: default
---
<div class="sectionHeading">Introduction</div>
<div class="section">
The goal of this site is to provide examples and articles that simplify the learning process of various information technologies. If you happen to have an example application, lesson, or article that you'd like to provide (credit given to you of course), please contact me.
<br/><br/>
One of the unique things about this site is that you'll notice several "CRUD" applications that all deal with the same concept of updating mock Employee information (CRUD refers to the typical application's dealing with create, retrieve, update, and delete actions.) The nice thing about these CRUD applications is that they all use the same simple in-memory persistence daos and the behavior and look-and-feel of all of the applications is the same. This makes it really nice for comparing how different frameworks handle the same type of CRUD concepts in a web application. 
<br/><br/> 
As the site grows, more different technologies will be added. Since technology moves so quickly, many of the examples here will be 
very outdated. Be sure to check approriate web sites for updated information concerning the technology you are investigating.  

<div class="sectionHeading">How you can help</div>
<div class="section">
Feel free to contact me with any mistakes you find in any of the lessons or articles (spelling, grammar, or content.) 
Also, if you have worked on a good lesson or example application and want to have it posted, let me know as well.
<br/><br/>
Maddi Rocks!
</div>

<div class="sectionHeading">Articles (most recent listed first)</div>
<div class="section">
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.date | date:"%m.%d.%Y" }} - {{ post.title }}</a>
    </li> 
  {% endfor %}
</ul>
</div>