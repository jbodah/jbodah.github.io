---
layout: post
title: On Being Impactful
---

I've felt impactful at a few roles in my life. At Stripe it feels like I'm still getting there, but I at least see a hazy road to success and it does not feel like a hopeless endeavor.

I think impact is relatively easy to measure. Most things in business can be boiled down to numbers, and those numbers can be tied back to dollars and those dollars can be used to compare against other dollars. The point is that dollars can be the common denominator of basically all changes and actions. To me impact of any change or action can be modeled as:

    (impact of change) = (duration of time change will be applicable) * (% chance of change/action being hit) * (diff in number of dollars)

A positive impact results in an increase in dollars, a negative impact is represented by a decrease in dollars.

For the sake of it, let's list out some examples:

* A change that is **applicable for a long time**, has a low chance of occurring, and has a low item-price difference → could be very impactful if we wait long enough
* A change that is applicable for a short time, has a **high chance of occurring**, and has a low item-price difference → could be very impactful if the chances are high enough
* A change that is applicable for a short time, has a low chance of occurring, and has a **high item-price difference** → could be very impactful if the price difference is high enough

In isolation, all of these have a questionable probability of being very impactful (note that "very impactful" implies that it is *more* impactful than other potential uses of time). However when we combine these then our confidence in high impact increases dramatically.

We have to admit however that it is difficult to know all of these things ahead of time. It may seem like we'll stick with a new refactoring for a long time, but we might decide in 6 months that we're going to rewrite it all anyways because of another problem. We may think that something has a high chance of occurring, but we fail to recognize that other changes greatly reduced to likelihood of our change occurring. We might think a new product is going to be high value and thus lead high sales prices, but it turns out users aren't willing to pay for it and we have to lower the price. We deal with each of these with various estimations of course, and - even when our ideas fail to be impactful - this is the right approach.

What I want to focus on though is the opposite side of the coin: recognizing changes and actions that will have low impact (and subsequently not doing them). I feel that recognizing low impact work is even more valuable than recognizing high impact work as low impact work is a waste of time (assuming you have the ability to take on other work, of course - low impact work can be a great learning tool).

Incident-driven work is a good playground for exploring this idea, and I think that's the case because incidents are often intense and blood-pressure-raising events. People are often unhappy that something happened the way it did and want to **prevent** it from happening again. I want to argue that this can lead to expensive low impact work and would like to stress how using a framework to direct all of our work (even work that is recent and stinging) is useful for ensuring we are reasoning rationally about what we choose to spend time on.

Let's take a simple example:

* Application XYZ had a 10 minute outage. On investigation it turns out that an operator had incorrectly configured the application, deployed it during off-hours, and did not immediately recognize the problem. Once the outage occurred, it took 15 seconds to recognize the issue. It took 2 minutes to deploy the rollback and reboot the application.

I'm sure you can already start rattling off ideas for changes we could be making here:

* The application was incorrectly configured. How did this happen? Maybe we should write a script to validate configuration prior to deploys
* The application was deployed during off-hours. Why? Maybe we should require all off-hours changes to be verified by two on-call engineers prior to being deployed
* We didn't immediately recognize the problem. Why did it take so long to recognize? Maybe we need to improve alerting to ensure we are quickly notified when things go wrong
* Our end-to-end rollback took 2 minutes. Why did it take so long? Maybe we need to rethink our deploy and rollback strategy

The naive approach here is to do all of these, and I couldn't fault anyone who felt they were all valuable solutions. I would agree that all of these could've prevented the incident or made the incident go smoother. I want to dig in further however and argue that I think two of these changes are pretty likely to be low impact.

* The first change proposes writing a validation script. How often do we see misconfigurations happen? Are they really so common to justify **both** the development **and** support of this feature over many years? Will it be worth the additional overhead it will cost us to run this in CI or development overhead it will cost us in process? I would argue that there is not a strong case for this to be high impact, and thus I feel this is a **low impact** proposal

* The second change involves adding additional process to off-hours changes. It's likely that we'll want to make off-hours changes in the future even though I'm not sure it'd be very high frequency. This process would likely last us many, many years and given it is a human process rather than a technical one it shouldn't atrophy as technology changes. How often do people deploy off-hours? When they do, how often do things break and how badly? I'm a little concerned that there isn't enough *evidence* that this definitely will be high impact. It's probably a good idea so everyone can have dinner with their families instead of working, however as we'll look at soon - this doesn't really feel like it is the **need** that the incident is calling for. Not enough evidence this would be high impact - thus, **low impact** for me

* We could improve our alerting so that we know more quickly when things go wrong. Assuming the operator was truly acting in the company's best interest, one can probably sympathize with the operator. We should know when things break, and we should know fast. Things will break again and we will want to know quicker in those cases too. Our company is growing and adding new devs all the time, so it's likely we'll experience this many times in the future. Faster failure detection is something we'll benefit from for a long time and would set an example for our other services. Moreover, the difference between ~8 minutes to recognize an issue and a few seconds could be massive considering our company's volume. This seems like a **very high impact** piece of work

* Our rollback took 2 minutes. It's likely that we'll need to rollback again in the future. We can look up our bad deploy rates to get an estimate for how much time per year this could save us and subsequently how many dollars that additional availability could save us. This work could also scale to our other systems too. This seems like it **could be high impact** - we'd like to do a bit more verification, but it is promising and the future wins are exciting

Incidents are the prime candidate for measuring impact, and I feel that incidents usually provide very clear measures you can use for justifying impact. Hopefully I helped demonstrate how you can triage proposals to avoid doing a lot of work that you never reap the rewards from. I don't mean to talk away all remediations either - sometimes it is the right thing to spend time on the spot fix or the bespoke answer, but usually it is not. I haven't even looked at the expense of solutions - that is also worth considering to make sure you weigh the **cost-benefit**.

There are other issues with time-pressured work that I haven't covered yet but are worth mentioning. Work that is taken on hastily is less likely to have edge cases well-thought through. What happens if your changes don't scale or you miss important cases that aren't caught for years? All changes present a risk too. You might be fixing system A, but you skipped out testing how it integrates with system B - hopefully breaking system B didn't create a bigger problem than your initial problem with system A.

The point is not to never make change - the point is that spending time on work that you could've preemptively predicted as low impact is a mistake, and it's a huge mistake when it has not only low impact but has negative impact (whether that negative impact be accrued immediately or over time). We should put a bigger emphasis on justifying **why** we are working on something and we we believe it does not fall into those trappings. We should also **challenge** our ideas to not only [estimate](2020/12/17/estimation/) the **ceiling** of our potential impact, but also recognize the potential **floor** - maybe we'll decide not to do something altogether, or maybe we'll find a reason to put even more resources into it if we can't find any holes.
