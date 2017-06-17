# Support and Troubleshooting Techniques for Software Developers

Developers at the company where I work, including myself, do a regular rotation as 2nd or 3rd level product support. Basically, whenever our regular customer support folks have support questions they can't resolve on their own, they reach out to the on-call developer. As you might expect, this is not our favorite part of the job, but sometimes it's rewarding. To it a little more bearable, sometimes I think of it as detective work. Think of a police procedural TV program like "Law and Order", where the detectives going around interviewing eyewitnesses to the original crime, which produces more leads, etc.  

I'm assuming at this point that a support person has come to you with a customer problem that they can't solve. At this point, your job is to go where support people generally can't - into the code.  

## Root Cause Analysis

Here's my main point: **Work Backwards.** Yes, I said this in the [last post] as well, and I'll repeat the rest of it: Start at the point where the error first shows up, figure out what condition(s) cause that to occur, then figure out what causes that, etc. Methodically work your way up the chain of causality. Don't attempt a fix without any analysis, unless you have a really good reason to believe it will work in this situation - in which case, you're not troubleshooting, you're just applying a remedy to a known issue. If support didn't know about the issue, be sure they add it to their knowledge base.  

## Start with some Basic Information 

I probably sound like a broken record to our support team, because I'm always asking for two things right off the bat:

1\. What version of the software is the customer running? If you have a single, web-based application that everyone in the world hits, this won't be a concern.

2\. Can you get me the logs?

I need to know what version they're on so I know which version of the code to pull to start the causality investigation. I want the logs because the logs usually tell me a lot more about the error than the user-facing error message does (and sometimes there's no user-facing error message at all, just a "something didn't save correctly" or "so-and-so didn't respond." If it's an exception thrown, the logs will give me the full exception stack trace, which is invaluable. Again, this is the way we tie the error to the code.

If your software does not log errors when they occur, stop right now and make it a high priority. Do not release your next version without it. If you encounter resistance on this, start looking for another job, because I can't think of many things more frustrating than having to support software that doesn't have a way to tell you when errors occur, and what the nature of those errors is.

Some other things to ask the customer (or have Support ask the customer):

*   If something had been working and suddenly started having problems, what has recently changed in the environment? New hardware, new software, version upgrades, etc.?
*   Is the problem isolated to one user? If so, start investigating what is different about that user. Insufficient permissions?
*   Is the problem isolated to a single workstation? If so, same thing - what's different about that workstation?
*   Is the problem isolated to a single piece of data? Perhaps some data got corrupted.

## Be Methodical

Don't be in a too much of a hurry - if you follow the steps, you'll almost always eventually come to the right answer. Apply logical reasoning: as a software developer, that's a big part of what you get paid for.

For example, if the user is seeing a particular error message, I need to know exactly what the error message says - preferably a screenshot or a text copy of the message. As a developer, your job is to tie the error back to the code. Sometimes you'll know exactly where in the code to look, but often, especially if your codebase is large, you'll need to search for the error text in the code.

## Static Analysis

Once you've found the line of code that produces the error message, start doing some static analysis. Use your IDE's "find usages" or "find references to" functionality (or in the worst case, the straight "find" functionality) and start going up the call stack. The possible call stacks may start fanning out, but if you cross-reference against log messages, plus what you know about the configuration of the system, you'll usually have a good idea of the code path. You may need to go back to the support team or the client to get more information.

In modern AJAX web applications, the error may first be thrown on the server, but the call stack will eventually lead to a web service call, and from there into client-side JavaScript. The same rules apply, although the functional, untyped world of JavaScript can make static analysis a bit more difficult.

If you can't find the error or exception message in your code, it's probably coming from a third-party library you're depending on. Next stop: Google (or your [favorite search engine](http://backtosquarezero.blogspot.com/2012/09/im-feeling-ducky.html)) or maybe [stackoverflow](http://stackoverflow.com/). Once you find the library function call involved, look for where it's called in your code, then follow the call stack up as before.

## OK, That Didn't Work...

If nothing has panned out so far, here are some other things to try. These are roughly in order of easiest-to-hardest.

*   Is this a known issue? Check your bug database, release notes, knowledge base, etc. Yes, support should have done this, but you probably have access to more resources (the bug database, for example). Also, with your perspective as a developer, you may search the knowledge base in a different way and come up with different (sometimes better) results.
*   If the error is repeatable but you might be able to bump up the logging level (turn on Debug logging, for example), then reproduce the error and get the new, enhanced log. The debug messages may tell you enough about what's going on to determine the problem.
*   Sometimes you'll need to look at the change history of code, to find out when something was broken (or was fixed). If your changesets are tied to tickets, ether enhancements or bugs, it will further shed light on why a particular change was made.
*   See if your testing team can repeat the error. If so, they'll probably enter it as a defect.
*   See if you can repeat the error in your development environment. If so, you can probably capture the problem in the debugger, which will give you much richer information that the static analysis approach detailed above.

## What To Do If You Get Stuck

*   Brainstorm the problem with your colleagues. They may remember a crucial piece of information you don't have, or come up with an approach you haven't thought of.
*   Relax your assumptions. If all assumptions were valid, the application would be working perfectly, right?
*   Some problems are really hard to diagnose because there are multiple problems interacting. Sometimes this is because there's a bug in the error-handling code
*   Remember that everyting is obeying physical laws, and there is a rational cause and effect going on. Do not treat the system as though it is magical.
*   That said, occasionally problems happen that you just can't explain. Hopefully this is very rare.

## Workarounds

If it's a problem in production, you may actually have two separate tasks: getting it working ASAP. This may involve manually fixing some data in a database, or it might require a workaround or temporary expedient of some kind. If the problem is due to a software bug, making sure a bug report is entered so it gets triaged, prioritized, and (hopefully) fixed at some point.

## Make It Easier to Troubleshoot Next Time

If the logging or other visibility into the code is inadequate, making the issue difficult to troubleshoot, the bug report may be "the so-and-so module has insufficient logging for troubleshooting and support," or better yet, something more specific, e.g. "The Flooper module doesn't log the full path of the file when it gets a 'cannot open file' error."

One thing I've learned about logging over the years is you definitely want to be able to log every significant interaction with the external environment: file opens, database interactions, web service calls, etc. You can't control these external things, but you can control exactly how you call them. In the worst case, these log messages could serve as a helpful bug report to the developers of the external module, but in most cases, you'll find the problem is with your own code.