
# Find and implement a better security model for Atom <br>

# Background

  * Name &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -  Andrei - Constantin Popescu
  * E-mail&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    -  popescu.andrei1991@gmail.com
  * Phone &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -  +40 0728 944 811
  * Studies completed &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -  Bsc. in Computer Science at Polytechnic University Bucharest
  * Studies in progress &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  -  Msc. in Computer Science At VU Amsterdam, starting this fall
  
  <p align="justify">
&emsp; Hello! I'm Andrei, I'm a 24 year old student from Romania, about to move to Netherlands. I'm passionate about technology in general, but specially about software! I picked up programming at 7 and since then I haven't really let it go. <br> <br>
&emsp; As an undergraduate, I tried to explore as much of the CS field as I had the opportunity to, as you will see my experience is  diverse.However, I'm a lot more familiar with low level/application level development than web development. So why not stick just to that? <br><br>
&emsp; I believe web has gotten too big to simply ignore it, unless you're doing something very low level and very specific. My belief is that Atom's mission and my vision aren't that far from each other. Should I say <a href=https://atom.io/packages/ask-stack> "Stack overflow integration" </a> ? I use Atom now and it's a product I do believe in. It has seamless git integration and I could find most plugins I used in Sublime  <br><br>
&emsp; Aside of coding, I'm a (mostly pick-up) basketball enthusiast, gym junkie(see my signature at the bottom) and outgoing person. I feel I'm a better engineer if I keep a ballance between social life and coding, so whenever I got the chance, I go out! After all, I can code at night as well, no?

</p>

# Project Idea

## Current situation
<p align="justify">
&emsp; Atom needs a better security model. Right now, an user is able to load a native module with malitious code which could potentially offer access to OS resources or other unintended behaviour. This might seem obvious, but I'm going to go through it anyway. The main <a href=https://github.com/atom/atom/issues/1763#issuecomment-82200875> issue  </a> isn't getting access to Atom internals, as it is open source and there's little gain in hacking an editor that's purposely made for hacking.  So, <i> no big deal for getting access in  Atom .</i><br>
&emsp; My understanding is that the <i> hot </i> issue is gaining access to operating system priviledges(in example, root) through Atom. This, in my opinion, is a valid deterrent. Let's picture a case where I would use Atom to edit my /etc/sudoers. I would do 

> sudo atom

With the atom process running in priviledged context, it's easy to see what could happen if malitious code would run inside a Node.js module.
</p>

## What we need

<br>We need a better security model!<br>
&emsp; It would take the form of a capability to restrict or at least supervise a module's accesses to OS resources. At worst, we need to make sure that we prompt the user to confirm, whenever something <i> untrusted </i> is to be executed. <br> 
&emsp; Aside of that, we also need to maintain the same functionality, without dumbing down or suffering 
observable performance loss*

> observable performance loss roughly means "you have to wait for the program to finish"

<br>
## Approach
<br>
<p align="justify"> The project will kick off with efford directing mostly towards isolating pure JS packages in their own context, other alternatives being researched concurently, but with less priority. </p>

### Isolating pure JS packages in their own context 
<br>

<p align="justify"> 

&emsp;    Basically, in non-strict javascript code, there are many ways to get access to the global object (window in browsers), making true isolation a very hard problem. Caja does some iframing trickery to patch this. <br>
&emsp;   The main plan of attack regarding Atom securit model involves using security features/constructs similar to those used in ECMAScript. While the main goal is, as stated above, isolating packages in their own context, it's difficult to outline an exact plan on how to do it.<br>
&emsp;   I'm not familiar with Atom internals and neither with Caja, and this is something I plan to take care of during community bonding period(basically, when I'm not supposed to write code) but since atom uses Node.js I expect it's package loading mechanism not to be a much different beast than <a href=http://fredkschott.com/post/2014/06/require-and-the-module-system>loading node modules with require </a>.<br><br>
</p>

#### General techniques I intend to use
<br>

<p align="justify"> 
Now, some might be already there, some might not be applicable but I will list some general techniques used in ECMAScript
</p>

* Objects as closures

<p align="justify">  The idea behind it is to implement a tamper-proof record of lexical closures that encapsulate state. The object is thus able to defend itself from unwanted changes. <br> <br>
</p>  
* Revocable function Forwarder

<p align="justify"> 
  Mechanism for temporal attenuation on a given reference. This is possible by replacing the default refference with a function forwarder(that's "given" permanently) but which's target can be altered(set to NULL). By setting the forwarder's target to NULL we revoke access to the given target. <br>
</p>
* Membranes

<p align="justify"> 
 Membranes are the generalised application of Revocable function Forwarder, it cuts out all references.<br>
</p>

* Other access atenuators

<p align="justify">
 Attenuator to make a file read-only by passing just it's content and size, instead of passing a pointer to the file structure. This would prevent mailicious writing the file.<br>
 Replaced "eval", no powerful references by default, each object can go down on the relationship tree(and back up to it's level) but never up<br>
</p>

* Membrane eval

<p align="justify">
 This looks exactly like what we'd need to do in order to isolate a js package in it's own context. The parent creates a compartment that it can revoke at <i>any</i> point. It's useful to point out that membrane not only cuts out all the external references to this object but(more as a side effect) doesn't allow it to reside in the memory either. Once the references are removed, garbage collector is called <br>

I found support for <a href=https://github.com/lukehoban/es6features#module-loaders> secure module loaders</a> with which new loaders can be constructed to evaluate and load code in isolated or constrained contexts. <br>. There are plenty <a href=https://github.com/ddrcode/node-secure>resources</a> on how to go about it in ECMAScript 5 as well. <a href=https://code.google.com/p/google-caja>Google Caja</a> also does this and it remains to be seen if it can be used. I also found <a href=https://www.npmjs.com/package/google-caja>this npm package</a> that has the google-caja sanitizer<br>
</p>

<p align="justify">
&emsp; I'm not <i> that </i> familiar with Caja and Atom internals, but I believe the community bonding period will be just enough, as I have researched, learned and understood everything in this proposal with virtually zero previous knowledge, in a much smaller time frame.
</p>

#### Applications of mentioned techniques
<br><br>
Attenuate access to `fs`.  Limiting access to `fs` is a reasonable approach. A module should not be able to read(or more importantly write) a file that it's not supposed to. So, instead of offering a reference to it through an attenuator.  <br> 

Supposing we load an erroneous module, with membrane we could cut all references to it and it will be eventually cleared by the gargabe collector <br>

These are simple/common cases that could be valid, the general idea is, however, to go for the minimum rights principle. To let a module use just enough Atom API to work, but not more. <br>




## Overview of other researchable alternatives



### Native Client (NaCl)

<br>
<p align="justify"> Native client relies either on the following phylosophy(processes)<br><br></p>

<img src=http://goo.gl/6d85Sh>
<br> <br>

or on this <a href=http://static.googleusercontent.com/media/research.google.com/ro//pubs/archive/34913.pdf> paper  </a>.  The paper describes how sandboxing a thread would happen, which is similar to our goal.However, it does this with the tradeoff of limiting thread capabilies heavily( to four).
</p>

####Benefits  
<br>

<p align="justify">
&emsp; NaCl is one of the very few available means of achieving thread-level security. It offers security at thread level and obviously at native level. For Atom, it would mean that each module is loaded in it's own thread. Any code the malitious module will execute will break at most the respective thread, which the main node process can restart or stop. Moreover, it wouldn't have access to any other resources than the main node process decides to offer. <br>
&emsp; Essentially, it sandboxes each thread, controlling communication and access between them via an implemented messaging interface(that relies on IPC). <br>
</p>
<br> <br>
####Issues
<br>

<p align="justify">

&emsp; In order to achieve what was described above, NaCl imposes major restrictions on threads. For example, it restricts thread access to four syscalls*, all of them being relatively harmless. In the case of chromium tabs, it makes sense. They don't have much to do with the operating system,browser handles whatever acctions necessasry.<br>
&emsp; For Atom, on the other hand, my opinion is that this will impose major limitations, break a large number of packages and cause an overall unpleasant experience to package developers. <b>Overall, no-go.</b><br>
&emsp; The other approach, would be to allow require() and native code through. While require() <i> could </i> be modeled to have some whitelisting, asking the user to prompt, on native code, there isn't too much to do.
</p>

* syscalls `read()`, `write()`, `exit()`, `sigreturn()`

#### Conclusion
<br>
<p align="justify">
&emsp; I have reached out to the Node community and what I learned is that the running node module  is not a process, you just run some code in a 'v8 context'. Each module in node is run in a v8 context, and node sets some context globals like `module` and `require`. If you need to allow `require` or native code it's not a viable choice. NaCl is an overall no-go because it's developed to sandbox applications(threads) with limited requirements and it has no mechanisms to limit access to what we seek. It's has binary behaviour. It either allows it or not.

&emsp; To put it simply, the Chrome-like sandboxing(each process being sandboxed sepparetely) is not suitable for Atom. The other alternative, sandboxing threads, isn't straight-forward at all, it might not even be possible as you need to allow stuff like "require". Require can be indeed rewritten to a more secure model, but, imo, that can only happen at JS context level, not at native code level.
</p>

### Using containers

<br>
<p align="justify"> This is the idea behind containers, in this specific case [Docker](https://www.docker.com/). They provide isolation, but through a very lightweight framework. </p>

<br>
<img src=http://blog.trifork.com/wp-content/uploads/2013/07/Screenshot_from_docker.io_about.png> <br><br>

####benefits
<br>

<p align="justify">
&emsp; Instead of running a file in node, you run a docker container which runs that file. Loading a module on each container limits that module's access to OS resources. Basically, it provides exactly what we were looking for.<br><br>
&emsp; But what about performance loss and scalability, let's have a look:<br><br>

<img src=http://image.slidesharecdn.com/kvmanddockerlxcbenchmarkingwithopenstack-140430063510-phpapp01/95/kvm-and-docker-lxc-benchmarking-with-openstack-50-638.jpg?cb=1399803932 width=70% height=70%>

<img src=https://s3.amazonaws.com/media-p.slid.es/uploads/arashkaffamanesh/images/674171/benchmark_docker_kvm_memory.png width=70% height=70%><br><br>

<p align="justify"> All in all, it's not bad, considering Atom is unlikely to load thousands or tens of thousands of modules. As a personal experience, during my internship at Freescale, we managed to get about 1000 running on a 2. something ghz CPU and 4 GB RAM memory. <br></p>
On short, it achieves what we needed at a reasonable tradeoff.
</p>

> Note - Docker does claim that their containers are portable, but since they rely on lxc, which is linux
specific I imagine there's a significant overhead on other platforms(windows/mac).


####Issues
<br>

<p align="justify">
&emsp; Docker(in fact, LXC containers) were not built for isolation. <i> LXC containers do not fully contain.</i> I've done quite a lot of research on this subject and what I found out is that the opinions are divided.<br>
&emsp; LXC containers are a fairly mature technology and there's certainly effort in the direction to fix this. There are plenty articles like <a href=https://blog.docker.com/2013/08/containers-docker-how-secure-are-they> this one </a>  which advocate them as being <i> quite secure </i>.<br>
&emsp; On the other hand, there is <a href=http://linux-audit.com/docker-security-best-practices-for-your-vessel-and-containers> this article</a> that's quite recent and well explained which showcases Docker issues.
</p>

####Conclusion
<br>

<p align="justify">
&emsp; Ultimately, the decision on taking this path or not belongs to the Atom community or development lead. What I have described above is a detailed, bigger picture. <br>
&emsp; From what I have read on that issue, this has already been done at cloud-level and I could implement/deliver Atom packages with Docker in form of bundles (.deb, .rpm) or some other agreed upon formats.<br>
Here is an approximative docker receipe <br>
</p>
```
RUN apk-install nodejs

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

COPY . /usr/src/app/
RUN apk-install -t build-deps build-base python \
  && npm install \
  && npm run build \
  && npm cache clean \
  && apk del --purge build-deps \
  && apk del --purge build-base

CMD ["npm", "start"]
```
<br>

* Kudos to Node.js community for supplying the docker receipe! I'm familiar with LXC but have never used docker. <br>

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

* 27 April - 11st May - research on atom module loader,establish security liablities, based on the proposed plan, research JS internals <br><br>
* 11st May - 25th May - further research on security issues, settling on proposed security fixes(tuning my proposed security plan until it's accepted by the Atom community)<br><br>
* 25th May - 8th June - have a complete clear security model imrpovement plan and begin implementation <br><br>
* 8th June - 22nd June - work on security improvements; research on NaCl techniques (tests various implementations and packaging) <br><br>
* 22nd June - 6th July - finishing up midterm eval submission, concluding on Nacl implementation possibility, having a countable number of security features implemented in Aton<br><br>
* 6th July -  20th July - Working on module isolation, upstreaming previous work, fixing any issues caused by security features that were added, functionality test work-in-progress <br><br>
* 20th July - 3rd August - research on Docker implementation possibility and coming up with a documented conclusion, further work on security model <br><br>
* 3rd August - 15th August - finishing JS security features, brushing up code <br><br>
* 15th August - 21st August - merging upstream, doing functional testing , fixing aditional issues, writing final report <br><br>


### Deliverables

* Accountable number of security features(i.e isolate fs, isolate buffer API)

* Paper on securing Atom modules with NaCl, with relevant conclusion, backed by facts.

* Paper on Docker implementation situation, with relevant conclusion, backed by facts.


#### Important Note
<br>
<p align="justify"> Even if in Timeline the testing, brushing up and merging are listed at the end, that's not the effective way it will happen. They're listed that way as I had to account for the given time. Each feature or fix that I add will go through the following flow <br>
</p>
```
implemented -> (functional and unit) tested locally -> pull request to main repo -> refine until accepted -> fix any broken issues into main repo, development branch. 
```
<br><br>

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
* Because I'm goal-oriented, driven and no stranger to taking risks. <br>
&emsp;&emsp; took a kernel developer internship knowing just pointer arithmetics <br>
&emsp;&emsp; Got involved in a project I had literally zero knowledge about and put enough effort to achieve a functional final result. <br>
* Because I got a relevant background, in both directions <br>
&emsp;&emsp; Low level knowledge from mentioned internships <br>
&emsp;&emsp; self-developed Node.js, javascript and web knowledge <br>
&emsp;&emsp; C/C++ as strongest language, experience developing C++ apps
* Because I believe in Atom as a product <br>
&emsp;&emsp; this is an important factor in my motivation and how much work I put into something.
* For the love of octokittens? <br>
 
<img src=http://venturebeat.com/wp-content/uploads/2012/12/github-open-source.jpg img width = 35% img height = 35%>

> Note: I do not own any credits for the photo above.
  
  
  
Thanks for taking the time to read this!

Regards, <br>
Andrei<br><br><br>
<img src=https://octodex.github.com/images/steroidtocat.png width = 25% height = 25%>
