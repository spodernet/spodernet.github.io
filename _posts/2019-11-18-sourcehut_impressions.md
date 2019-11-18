---
layout: article
title: First impressions on Sourcehut
tags: sourcehut git
---

Sourcehut recently marked the [1 year anniversary](https://sourcehut.org/blog/2019-11-15-sourcehut-1-year-alpha/) of
its public alpha release, and what better circumstance to share my initial thoughts on it?

I've started using Github in early December of 2011 but I was never satisfied with it, I can't put a finger on why,
even though I had a pro account (thanks to being an university student), but alas, I began hopping from a git hosting 
service to another. First I migrated to Bitbucket, I gave it a spin but again, I wasn't really satisfied. I feel like
they have too many features which I don't care about, and the ones I cared about didn't work very well for me. For 
example I didn't like that the free plan only included 50 minutes a month of pipeline usage and the LFS storage is 
limited to 1GB per month, after which you can't push anymore. Sure I could have paid <span>$</span>3 a month to get 
2500 minutes of build time and 5GB of storage, but ultimately I didn't want to support a closed source service and 
their pricing model. Fast forward a bit and I switched again, this time to Gitlab and after the Github acquisition by 
Microsoft, I moved most of my projects over to Gitlab. However, even though I consider it an improvement over the 
previous ones, again I felt like it wasn't for me. Gitlab, Bitbucket and to some extent Github all feel like having an
overly complex interface, and in the end it ended up getting in the way of me being productive.

One day I was being productive, by browsing the internet of course :) when I stumbled upon a beatiful tutorial on [git
rebase](https://git-rebase.io/), and something caught my eyes, it was these 3 sentences: "_This guide brought to you 
courtesy of sourcehut, the hacker's forge. 100% open source Git & Mercurial hosting, continuous integration, mailing 
lists, and no JavaScript! Try it today!_". I thought, "_mmh, an open source git hosting service with no JS? I've gotta
take a look_", so enter sourcehut.

## Sourcehut
Right after discovering sourcehut I decided to make an account and try it. After about two weeks of usage here are my
impressions. Disclaimer: at the moment sourcehut is still in alpha, so things might change.

- **Services**: sourcehut currently offers hosted git (and mercurial) repositories (public, private and unlisted), 
issue trackers, mailing lists, a build service that let's you compile and test code, deploy websites, build and 
publish packages and a task dispatcher to automate taks and integrate sourcehut with other services like Github. 
The best thing about these services is that they are decoupled from each other, you can have a git repository and no issue
tracker or an issue tracker without having a git repo, not only that but all these services can work in conjunction
with each other and in an automated way, awesome. Some people may not like this feature division, but for me it's actually
perfect;

- **workflow and user experience**: the decoupled nature of its services together with the clean interface is a godsend
for me. I dislike the overly complex interface full of menus, submenus and submenus of submenus that some services have.
Sourcehut (interface) let's me focus more of what I need to do and not on where I have to click to access what I need.

- **pricing model**: the pricing model follows a "pay what you want" strategy and it's divided in 3 tiers (in terms of
price): <span>$</span>2/mo or <span>$</span>20/yr, <span>$</span>5/mo or <span>$</span>50/yr and <span>$</span>10/mo or
<span>$</span>100/yr. For now payment is optional, but when it reaches the beta stage users that want to have access to
all the features have to pay. If you don't want to pay you can still contribute to project hosted on sourcehut. I 
really like this approach because it let's the user decide how much he wants to pay for it, and did I say they all 
offer the same features, and that you can earn free service if you contribute patches to upstream projects? 

- **transparency**: this is another strong point, I really like that Drew (the author) is always open to suggestions
and keeps an open line of communication with the community, not only by having a dedicated [mailing
list](https://lists.sr.ht/~sircmpwn/sr.ht-discuss) but by posting financial reports and "what are we up-to" style 
[posts](https://sourcehut.org/blog/). It makes me feel like being part of the development and overall evolution of the 
platform;

- **no JS and FOSS**: the icing of the cake is that it's 100% Free Software and all features work without any javascript.

But it's not all nice and shiny and being a new platform also means having some issues, for example in my first week of
using it I noticed some, how can I say it, less than optimal performance when performing git commands over SSH.
Sometimes it would take more than 10 seconds to perform a simple fetch or to push some changes. Fortunately things are
getting better, but there's still some improvements to be made and they are working on it.

## Conclusions

All in all, things are moving relatively fast and as the 
[last update post](https://sourcehut.org/blog/2019-11-15-sourcehut-1-year-alpha/) says, we are near the beta stage and 
all the services are coming together to create a complete system. In conclusion I'm really excited in what's coming for
sourcehut and I'll probably write a full review once it's mostly done.
