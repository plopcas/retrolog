---
title: "Mastering Pair Programming in 5 Minutes"
date: 2020-03-08T16:41:21Z
draft: false
tags: ["techniques"]
author: "Pedro Lopez"
---

![image](/images/mastering-pair-programming-in-5-minutes.jpg)

This post is heavily based on https://martinfowler.com/articles/on-pair-programming.html (38 min read). That is, however, a very long (yet very good) article. For those of us that can't or don't want to afford spending that long, I collated its main points and added a few twists of my own. I hope it's useful.

<!--more-->

####  Why bother?
According to some studies e.g. https://collaboration.csc.ncsu.edu/laurie/Papers/XPSardinia.PDF, some of the most significant benefits of pair programming are:

- Many mistakes get caught as they are being typed in rather than in QA test or in the field (continuous code reviews)
- The end defect content is statistically lower (continuous code reviews)
- The designs are better and code length shorter (ongoing brainstorming and pair relaying)
- The team solves problems faster (pair relaying)
- People learn significantly more, about the system and about software development (line-of-sight learning)
- The project ends up with multiple people understanding each piece of the system
- People learn to work together and talk more often together, giving better information flow and team dynamics
- People enjoy their work more

The development cost for these benefits is not the 100% that might be expected, but approximately 15%. This is repaid in shorter and less expensive testing, quality assurance, and field support.

#### How to pair
When the task involves coding, three options:

* **Driver and Navigator**
  - Driver is the person with the keyboard focused on completing the immediate goal at hand, temporarily ignoring larger issues. Should talk through what they are doing while doing it.
  - Navigator is the observer while the driver types, reviews the code on-the-go, gives directions and shares thoughts. Keeps an eye on the larger issues, potential bugs and makes notes for next steps and obstacles.
  - Driver thinks **tactically** and is detail-oriented while the Navigator thinks **strategically** and has the bigger picture in mind.
  - Tips:
    * Start with a reasonably well-defined task
    * Agree on one tiny goal at a time e.g. defined by a unit test or a commit message
    * Switch roles regularly
    * As Navigator, avoid tactical thinking, you'll be tempted! Make notes on potential obstacles and ideas and discuss after the tiny goal is done, so the Driver can stay in the flow
* **Ping Pong**
  - **TDD** oriented
  - Ping - Person A writes a failing test
  - Pong - Person B writes the implementation to make the test pass
  - Ping - Person B writes a failing test
  - Pong - Person A writes the implementation to make the test pass
  - ...
  - Tips:
    * Start with a clearly defined task
    * Each Pong can be followed by a code refactoring step done together by both (Red - Green - Refactor)
* **Strong-Style**
  - For an idea to go from your head into the computer, it MUST go through someone else's hands
  - Particularly useful for **knowledge transfer**
  - Driver is a novice
  - Navigator is much more experienced with the task at hand
  - Participants **do not switch positions** throughout the session
  - Tips:
    * Driver totally trusts the Navigator and should be comfortable with incomplete understanding
    * "Why" questions and challenging the solution should be discussed after the implementation session
    * Don't overuse this technique
    * Great for initial knowledge transfer but the goal is to transition to the other styles after a while

When the task does not involve coding.

* **Planning** - what's our goal?
  - E.g. pairing on Epic Ownership or aspects that overlap multiple epics
  - Understand the problem, read through the story and play back to each other how you understand it to make sure there is alignment
  - Clear up any questions or potential misunderstandings with the Produc Owner / Engineering Manager / Epic Owner
  - Double check your Definition of Ready to make sure you have everything to get started
  - Brainstorm a potential solution, together or individually and then present ideas to each other. If one is more familiar with the domain or technology, take some time to share context.
  - Plan your approach, list out the steps to get to the solution you chose and how you will test it. Write them down! so that we can track progress (e.g. in JIRA as subtasks)
* **Spikes** - research and explore
  - Define a list of questions that you need to answer
  - Split up, divide and conquer. Alternatively, try to find answers for the same questions separately.
  - Search your company's intranet and messaging system, and any other resources you can find
  - Read up on concepts that are new to both of you
  - Get back together after a previously agreed upon time-box, share and discuss
* **Documentation**
  - Reflect together if any documentation is necessary for what you've done
  - Create a document together. Alternatively, one person creates it, then the other reviews it and "wordsmiths".
  - Pairing reduces the chance of skipping documentation

For time management you can use the pomodoro technique, small breaks after 25min and longer breaks after 2h.

Pair rotations:

- Don't stay in the same pair for more than 2-3 days
- This avoids silos, increase collective code ownership, and favours more code review on-the-go
- It also keeps things fresh, particularly when you are working on something tedious and energy-draining
- The person who stays is called the Anchor

Plan the day:

- Pairing requires a certain level of scheduling and calendar coordination
- Start the day by agreeing on how many hours you are going to pair, and whether you need to plan around meetings or other commitments
- If someone is off-sick or on leave, rotate the pair
- Consider leaving some time at the beginning/end of the day for other activities e.g. checking your email or self-studying

Physical setup:

- Make sure both have enough space, chairs included, clear up the desk if necessary
- Decide if you want to use one keyboard or two, same for the mouse. No hard rule, whatever works. Some other factors into play are hygiene and how much space is actually available.
- Consider using screen share on Zoom so that each one can use their own laptop
- Agree on a default the first time, so that you don't have to discuss this every time
- Remote pairing is ok as long as the internet connection is good enough, there is no background noise (use mute otherwise), and both of you agree to it.

Celebrate when you have accomplished a task together, you earned it!

Things to avoid:

- Drifting apart during the pairing session e.g. reading emails or using your phone. These can come across as disrespectful and distracts you from the task at hand. If you really need to check something, make it transparent, say what you are doing and why
- Micro-managing e.g. "Now type this...", "Now we need to create a class called...", "Press Cmd + Shift O..."
- Impatience, apply the 5-seconds-rule instead (when the Navigator sees something "wrong" and wants to comment, wait at least 5 seconds in case the driver already has it in mind, so that you don't interrupt their flow unnecessarily)
- Keyboard hogging, are you controlling the keyboard all the time? Sharing is caring ❤
- Pairing 8 hours per day, it's not sustainable, it's too exhausting and in practice there are other things you need to do other than coding

There is not "THE" right way, try different approaches and find what works for you and your team.