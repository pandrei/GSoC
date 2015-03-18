
# <p align="center"> <font size="6">Find and implement a better security model for Atom</font> </p>

# Background

  * Name &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -  Andrei - Constantin Popescu
  * E-mail&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    -  popescu.andrei1991@gmail.com
  * Phone &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -  +40 0728 944 811
  * Studies completed &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -  Bsc. in Computer Science at Polytechnic University Bucharest
  * Studies in progress &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  -  Msc. in Computer SCience At VU Amsterdam, starting this fall
  
  <p align="justify">
&emsp; Hello! I'm Andrei, I'm a 24 year old student from Romania, about to move to Netherlands. I'm passionate about technology in general, but specially about software! I picked up programming at 7 and since then I haven't really let it go. <br> <br>
&emsp; As an undergraduate, I tried to explore as much of the CS field as I had the opportunity to, as you will see my experience is  diverse.However, I'm a lot more familiar with low level/application level development than web development. So why not stick just to that? <br><br>
&emsp; I believe web has gotten too big to simply ignore it, unless you're doing something very low level and very specific. My belief is that Atom's mission and my vision aren't that far from each other. Should I say <a href=https://atom.io/packages/ask-stack> "Stack overflow integration" </a> ? I use Atom now and it's a product I do believe in. It has seamless git integration and I could find most plugins I used in Sublime  <br><br>
&emsp; Aside of coding, I'm a (mostly pick-up) basketball enthusiast, gym junkie(see my signature at the bottom) and outgoing person. I feel I'm a better engineer if I keep a ballance between social life and coding, so whenever I got the chance, I go out! After all, I can code at night as well, no?

</p>

# Project Idea
<p align="justify">
&emsp; There currently is no clear, obvious way of implementing  native-module security. As a result, the least I can achieve is a highly documented and detailed proof of concept of why it can't be done, or the issues I ran into, with fallback on JS security

from  <a href=http://static.googleusercontent.com/media/research.google.com/ro//pubs/archive/34913.pdf>Native Client: A Sandbox for Portable, Untrusted x86 Native Code </a>

> <p align ="justify"> 
 Native Client is ideal for application components requiring pure computation.<br> It is not appropriate for modules requiring process creation, direct file system access, <br> or  unrestricted access to the network 
 </p>

My observation is that you can't  secure threads from one another, at least not without major restrictions. NaCl does this, but it limits functionality to 4 calls (read, write, exit and sigreturn, so no malloc).

What I propose is to head the research in the following direction <br>
<img src=http://goo.gl/6d85Sh> <br>
<p align="justify"> While it's very difficult to implement security at thread level, at process level the operating system helps. So a module would be loaded in the following manner:

 load would spawn a new process(for the module) -> the main node process would freeze(until the module loads). -> main process would provide information via IPC.
 Looking at <a href=https://github.com/atom/atom/issues/1763#issuecomment-82200875> this issue</a> there is a debate about the filesystem security.
  </p>
  > If there's no access to 'fs', significant packages break.<br>
  > If there is access to 'fs', all bets are off. <br>
  
<p align="justify">  
  My (probably overenthusiastic, irresponsible) impression is that those two don't mutually exclude each other.
  With the process-to-process communication idiom we could share (whitelisted) descriptors, this is known and
  has <a href=http://stackoverflow.com/questions/2358684/can-i-share-a-file-descriptor-to-another-process-on-linux-or-are-they-local-to-t> been done</a>.  <br><br>
  
  Is it perfect? No, but it gives a decent benefit for both cases.
  
  The only downside I can think of is a somewhat performance decrease, but since you're not going to load packages <i> that</i> often, it seems reasonable to me.
  
</p>

###Project idea roadmap

* investigate the possibility to load a module in a different process, in case of success begin implementation/design. <br><br>
* <p align="justify"> investigate using NaCl to have this at thread level. More difficult, as described here <a href=https://www.cr0.org/paper/jt-ce-sid_linux.pdf>paper on linux sandboxes </a>, you need to have two "parts". A trusted, secure runtime part where you can run basically anything, and a part where you limit functionality to a bare minimum.  For our purpose, this implies Having a "secure loader" part which has direct access to resources, and an "unsecure" one which interracts with the module itself. <br> </p>
* if all else fails, resort to <a href=https://github.com/gf3/sandbox>focus solely on the JavaScript layer and isolating pure JS packages </a>






First of all, this paper does not proppose a clear, obvious way of implementantion security using NaCl.
It presents what I have found so far, how I intend to continue research and what actions will I take based on the mentioned research. The fallback option is a JavaScript layer and isolating pure JS packages in their own context.

<a href=http://gf3.github.io/sandbox>This</a> looks like a good starting point! As well as <a href=https://code.google.com/p/setuid-sandbox> this</a>. 
</p>

References: <br>

*  <a href=https://code.google.com/p/chromium/wiki/LinuxSandboxing> Linux Sandboxing in Chromium </a>

# Project implementation & timeline
<p align="justify">
I believe it's quite obvious that while I do have some relevant background, you can clearly see that this project is a bit beyond my current level, and that's good, really! That's also why I apply for it!<br>

Based on the above, it's difficult to propose a clear implementation, instead, I'll propose a general flow.

If I discover a feasible way of implementing the security model, I'll tackle this in an "Agile" manner, sort of. Implement the easiest-to-implement priviledge restriction to get a hang of it, then prioritise the most important ones(i.e. fs access) first.
Since I'll most likely use Atom for this GSoC project, my implementation would get a bonus alpha testing already, so when I push it back to your main repo, there's a <i> minimum guarantee that it works </i>

Ideally, at least the first proposed research subject(using diff processes) would be completed during community bonding, coding period kicking off with either a plan for implementation(if research proves feasible) or with one less research direction. <br> <br>

All in all, this is, imho, a research project, so it's not possible to propose a clear, set scope but rather adjust it on the go. What I can guarantee is that Atom will get a significant contribution out of it,it'll be either having a better idea of what can and cannot be done or a better security model.

</p>

<p align="justify">

Note: by research I <i>don't mean reading stuff and storytelling</i>, I mean providing blog posts(they'll be on andreipopescu.net, I'll set up github pages with Jekyll on my domain) but rather docummented articles, with
code snippets as proof for why or why not that happens. I believe this approach makes them easier to discuss, read and gives a better exposure. I'm really open to investigating idea, provided there's some basis behind them.
</p>

###Timeline


I will be employed until 1st of June, working a 40 hour shift so my time availability is the following: <br><br>

* 27 April - 25 May - 20 hours / week <br> <br>
* 25 May  - 1st June - 20 - 30 hours / week <br> <br>
* 1st of June onwards - 40+ hours / week <br> <br> 

I divided my whole proposed timeline in intervals of two weeks. <br> <br>

* 27 April - 11st May - research on atom module loader, community discussions mixed with research about process spawn technique <br><br>
* 11st May - 25th May - further research on process spawn security model,best case - conclusion reached <br><br>
* 25th May - 8th June - last part of research regarding proces spawn security model, conclusion reached(mandatory) <br><br>
* 8th June - 22nd June - research on loading modules using NaCl <br><br>
* 22nd June - 6th July - finishing up midterm eval submission, getting to a conclusion on how feasible NaCl really is. <br><br>
* 6th July -  20th July - Provide an implementation design(based on the research above) reiterrate it a sufficient number of versions until community agrees. Provide enough arguments backed up by technology, with practical basis for the Atom community to be able to decide on a course of action and begin implementation(mid second week) <br><br>
* 20th July - 3rd August - Implementation of the respectively decided approach <br><br>
* 3rd August - 15th August - finishing implementation, code cleanup, merging upstream. <br><br>
* 15th August - 21st August - fixing aditional issues, writing final report <br><br>




# Previous experience

<p align="justify">
&emsp; I started out by being a Google Summer of Code student for <a href=http://brlcad.org/wiki/User:Popescu.andrei1991/devlogs2012> BRL-CAD in 2012</a>, my project involved doing benchmarks and functional testing a BRL-CAD networking library responsible for high-performance large-scale transfer(files up to 5GB). Based on tests' results, I had to identify the bottleneck (software or infrastructure wise) and provide improvements. <br><br>
  
&emsp; After GSoC, I remained involved with BRL-CAD as a Google Code-in mentor(past three years, including this one) and as an occasional contributor. This year I was significantly involved as a mentor, doing more than a quarter of total work.<br><br>
  
&emsp; I am and I remained involved with BRL-CAD as a Google Code-in mentor(past three years, including this one) and occasional contributor. This year I was significantly involved as a mentor, doing more than a quarter of total work.<br><br>
  
&emsp; Next, I headed to the low level field, as I was curious how things work, "what happens when I call malloc?". I took a 6 month internship to Freescale as a kernel developer intern, where I worked mostly on the kernel memory managent and just a bit on virtualization(hypervisor). Apparently, one of my patches made it upstream, as you can see <a href=http://git.freescale.com/git/cgit.cgi/ppc/sdk/hypervisor/hypervisor.git/commit/?id=b1a3966ab7c8dd13b53d66a3c1c6b9713bc1c2d3>here</a> <br><br>
&emsp; I was offered a place as a volunteer at an academic project called <a href=https://www.wyliodrin.com> Wyliodrin</a>, which  offers a web UI for boards control. Instead of working on low level, I decided to take the Node.js part as I had just heard about Node, was cutting edge technology. I had no clue about web, javascript or backend flows, I simply jumped into it hoping it'll hold. My task was to implement a Authentication system and I went with PassportJS and mongoose. Aside of that, I connected the authentication module to the main application.<br><br>
     
 <b>Relevant note:</b> In a way, there are a lot of similarities between my Wyliodrin experience and this proposed project. I wasn't an expert, I wasn't even average, but I pulled it through. <br><br>
   
&emsp; During my fourth(and last year) I took the Compilers and Kernel development courses which consolidated knowledge gained during the Freescale internship. The courses, on top of what I already knew, helped me create connections and really understand a general idea of how an OS typically works.<br><br>
&emsp; Following this, I succesfully completed another GSoC at BRL-CAD, <a href=http://brlcad.org/wiki/User:Popescu.andrei1991/proposal2014> extending a geometric kernel</a> they have. Basically, I did <a href=http://brlcad.org/wiki/User:Popescu.andrei1991/devlogs2014>class design and implementation</a> for a procedural library. The library itself is powerful, but inconvenient to use because of it's procedural nature.
  
&emsp; Lastly, I got employed as a C++ app developer in VoIP range, I'm currently learning javascript and node.js in my spare time. <a href=http://www.codecademy.com/en/tracks/javascript> Javascript tutorial</a> Some others I mentioned to <a href=https://github.com/nathansobo> Nathan </a> . <a href=https://github.com/pandrei/nodeauth> Some simple personal project </a>, <a href=https://github.com/pandrei/ISI-project/commits/master> another simple personal project </a> <br><br>
  
  To sum up, I'm a C/C++ developer and wannabe web dev. 
</p>

# So, why me?
* Because I'm goal-oriented, driven and no stranger to taking risks. 
> took a kernel developer internship knowing just pointer arithmetics <br>
> Got involved in a project I had literally zero knowledge about and put enough effort to achieve a functional final result. <br>
* Because I got a relevant background, in both directions
> Low level knowledge from mentioned internships <br>
> self-developed Node.js, javascript and web knowledge <br>
> C/C++ as strongest language, experience developing C++ apps
* Because I believe in Atom as a product
> this is an important factor in my motivation and how much work I put into something.
* For the love of octokittens?
 
<img src=http://venturebeat.com/wp-content/uploads/2012/12/github-open-source.jpg img width = 35% img height = 35%>

> Note: I do not own any credits for the photo above.
  
  
  
Thanks for taking the time to read this!

Regards, <br>
Andrei<br><br><br>
<img src=https://octodex.github.com/images/steroidtocat.png width = 25% height = 25%>
