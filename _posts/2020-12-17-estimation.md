---
layout: post
title: On Estimation & Confidence
---

One of the best interview questions I've ever had was also one of the strangest, and it has taken me many years to appreciate the question. The question was:

> "Our company makes those little bottles of shampoo you see in hotels. How many bottles of shampoo should we produce next year?"

Now, if you're anything like me you might be thinking "I have no idea, and I don't care - this is a stupid question", and that's totally fair, but there are some important subtle things at play that we will get into. For now, take a few minutes and just think about the question and what your answer would be.

You might have some follow-ups:

* How many hotels do we service?
* How many bottles did we produce last year?
* How many hotels even exist?
* How many people even go to hotels?
* How many bottles went unsold last year?
* etc, etc

These are all well and good and you should be asking these kinds of questions, but at some point you need to just answer: "I think we should produce X bottles".

Let's dig into a possible specific reasoning. You might come up with a basic model:

    (num bottles) = (num countries) * (num hotels/country) * (num rooms/hotel) * (num bottles/room) * (num days room in use/room) * ... etc

That's what I did at least, and it worked okay-ish. I spoke with some of my more clever colleagues who also were in the interview pool, and they told me a different model they'd used which I found eye-opening:

    (num bottles) = (num people in world) * (num vacations/person*year) * (num days/vacation) * (num bottles/day) * ... etc

That seemed _better_ to me - but I couldn't really explain why. As I dug into it I came up with the thought that my approach had more **potential error** in each of the rates - it felt harder to calculate each of these, and there were a lot of variables to account for and figure out. I felt like I had a better grasp of the variables in the second approach and thus it felt more trustworthy.

The two examples above illustrate two approaches to estimation: the first is a **bottom-up** form of estimation, and the second is a **top-down** form of estimation. In bottom-up estimation we are building our set from a small, tight number of things - including more and more as we go. In a top-down estimation we are shedding our set, filtering out more and more as we refine our estimation.

So you might be thinking: "oh, so top-down is more accurate" or you might think maybe I just didn't do a good job representing a bottom-up estimate (totally fair). I want to argue that neither of those viewpoints is the correct way to estimate something.

I think the correct way to estimate something is to provide **both** an independent bottom-up and top-down estimate; then you can calculate the **difference** between your estimates. If your estimates are close - congratulations, you probably have a pretty good estimate. If they are far off (say an order of magnitude or more) then you're clearly missing something. You need to add more details to at least one of your estimates. This difference gives you a **band of confidence** - an upper and lower bound of how sure you are. You effectively get a **clamping** or **pigeonholing** effect.

This is an extremely powerful tool when it comes to justifying decisions for which you have imperfect data too. Let's say you're not sure how many rooms hotels have. You might instead phrase the problem as a boundary value problem: "probably on the low of 30 rooms per hotel and on the high of 200 - who knows!". That is a perfectly fine methodology, and it will turn your upper and lower bounds into upper and lower ranges. Maybe they even intersect which might make you very suspicious that the real answer lies in that intersection. You won't always be right, but you're **being data-driven** - your informing your decisions with science and mathematics and it's this persistence and rigor that really counts.

You might be thinking: "aha, but what if I'm wrong or I completely miss some variable". That is valid too. In fact, the best predictors in the world tend not to be simple - nor do they tend to be complex. Instead they are **many** and **independent**. What's the best way to predict how many bottles of shampoo you'd need? Find 200 ways to predict it and aggregate their results. This approach of coming with a **matrix** of predictors gives you many knobs which you can tune and perfect over time. This technique comes up in many different places, but one of my favorites is machine learning where it's called **ensemble learning** - the idea being that it can be a lot more powerful to run many different types of models, compose their results, and try to iron out their aggregate error than to try and create a single model that perfectly solves the problem. If you're interested in learning more, I've found examples of this come up in proofs for algorithmic analysis and real analysis as well.

There isn't a limit to how far you can take this concept, but I do feel like there are quick diminishing returns unless you can take advantage of the margins (like in machine learning where small percentages can really pay off). For the lay-person such as myself, simply providing both a top-down and bottom-up estimation to clamp your estimation winds up being good enough for justifying most gut decisions.
