---
title: "Data Science is more than just Data & Science"
date: 2018-11-15T11:18:00+02:00
draft: false
image: "img/portfolio/"
tags:
- cloud
- edge
- UX
- data science categories:
- post
categories:
- post
description: "What I have learnt from deploying data science analytics for a customer"
---

# Context
Today's article will be very different from what I put on my website so far. As
I am writting this I am not even sure of the exact content yet, but let's start anyway.
So I've been working for more than one year part-time on a data science project for a
customer manufacture plastic parts. It does all the molding and the painting.  Those parts
are not very cheap to produce and there was a lot of scrap.  As result, they hired us to
find a way to reduce this scrap rate and save money of course. We've discovered other
things in this project that weren't exactlly part of project description.  In the end, we
should have been more cautious about our prerequites and constraints.


# Design of the solution
In terms of complexity it is between mediumish and hard, papers
we used are fifteen years old on average,  and honnestly we didn't use fancy neuron
network. The choice made one year ago is ok, but we are facing difficulties today that
could have been avoided using more *modern* methods. So couple of meeting with the
customer later, we realise that cloud was not an option, what will be delivered and tested
must run in the factory IT room to avoid data leackage. I've been doing mostly cloud for
years now, and edge only scenario are not my favorite one.  I am not talking here about
sensors and IOT stuff, but more of fog like setup, with a medium-sized server, that need
to be installed somewhere. First speach of wisdom: Do not let your customer drive the
deployement strategy, always push for the solution that make senses. In that case the
issue was as usual a false belief around data protection and privacy in cloud
environement.  As result, we end-up installing a single node server, with a bunch of
containers, no redudancy, no backup, and no vpn. Yes _no vpn_ access, If something goes
bad, you are good for a commando mission: plane, car rental, live debbuging, the fun
stuff.  Second speech of wisdom: If you really need to have an edge part in your design
_Always_ setup a VPN or at least to get to health status. The fun fact is that corporate
IT tends to be super slow for this kind of request, this can be a good solution to get
some extra time if you are running late. What I mean by that, is simple if you mess up
with your planning, allocate some times of your team to talk to corporate IT security
team, it's a treat. You will end with a bunch of requirements likes: os hardening,
firewall setup. As you always do that as part of any deployement model, you just bought
yourself a couple a week of extra on the back on your customer. It is a very nasty
trick, I agree, but it is a good thing to know. *Note* At the end of the interview with
the security folks gather all those recommendations and try to find value in it, for you,
new requirement, new security mitigation. What the application look like anyway: we
have couples of containers at the edge: UI, DB, EventBus, Compute nothing fancy here,
but the data ingestion part was a bit painfull to do, a mix shell, perl, python.
So some technical subject have been evaluated for the backend, but we also have a frontend,
 so we need a UI then.

# Your user is not who you think it is !!!
Here is what happened:

_Grumpy Dev_: Hey Ux Guy please do me a UI mock up!!

_Classy Ux_: Sure what for ?

_GD_ : I have a simple app to develop, some kpis to show and couple of graph

_CUx_: Ok, why showing this ?

_GD_ : Well the python lib will compute that and I need a display

_CUx_ : Who will use this app ?

_GD_ : I do not know !!

_CUx_ : Can we talk to him ?

_GD_ : I do not know !!

_CUx_ : I think we need to investigate usages, and practices here.

_GD_ : I just need a mock-up, I need it this week.

At this point you probaly understood where I am going...
Yes it's true, we didn't do any persona identification, we didn't meet the final user,
and above all, what was the problem we are told to solve in details.

For a datascience project it's a bit strange isn't it.
We actualy only know that we want to reduce scrap during the painting phase.

So we focused only on Data dictionnary: the thing that you do need to understand, the schema and logic behind it. We Thought it was alright so let's code my friend.

So this project went several month like that, all about coding, and not really delivering
value for the end user.

# But who is the user of this product ?

We installed the server in the IT room 3 months late for a 3 months development effort this
is significant. We started the evaluation after that, and everything was alright util some
design flows emerge, still no real final user.

After nearly 6 months of evaluation, we start talking about results, and possible extension
to other plants. How much money did we saved ? Does the quality improved after using the
analytics results? We discover that nobody was using the app, and nobody understood the
results it was giving. We talked to the head of digital, the guy that hired us, and we finaly
manage to talk to the right person, and define the usage with him.

It was too late! and soon after this discussion we discover other things.

The _real_ user at the end is the plant quality manager, who review on a weekly basis lines performance and define tuning of various machine to improve the performance of the line.

The ideal application was meant to be used at one of many starting points for the weekly staff meeting with all the various production engineers. Forecast curve was confusing and having a horizon of one day is not obvious to be seen as a forceast. Contributing parameters where not sorted by contribution weight. In the end we got many feedbacks for our application 2 months before ending the project.


# We don't know what happen !
As a final attent to prove our value, we put in place a massive set of calls to review results week by week.
After a couple of weeks, it was obvious that the analytics was not delivering upon expectation. The feature engineering was weak, and the forcast was overfitting quite a lot. The only thing that was satisfaying is that fact that we were able to tell with a very high probability wich parameters were causing scrap. Talking with process engineers was kind of frustrating, as we end up stating obvious things for them, based on our data analysis.

In the end the only thing we have prooven was our understanding of the process and all things that influence scrap.

# Final words, How does it end ?

We didn't sign any other contracts nor deployed our system to other plants. It's sad for my bonus indeed, but I do not concider this project as a failure at all as we learn so much on this project.

Here is a couple of points that I put on my checklist for any projects involving data science:

* Do not ruch on papers and data cleaning
* Try to have a UX state of mind :
  * Who are you working for ?
  * What is really expected ?
  * When your delivery will be used ?
* Having a clean and clear UI is more important than having the best model
* Having a way to monitor your model performance against real data is a must
* Timebox if you fail, move one, burning cash is never a good things

One side effect of this project for our customer has been a forced awareness of what being digital means.
We did quite a lot for them a side from pure datascience work, we help them realise lack of data gouvernance in plant and also the lack of controls in the overall paint process.

We haven't been paid for that, but we delivered all those benefits as a side effects of our work, maybe it's also important to bill things differently sometimes.
