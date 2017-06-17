# Troubleshooting Techniques for Software Support

In this post, I'd like to talk about some basic troubleshooting skills for software support people. If you're such a support person, I'm assuming you can apply some basic logic to a problem, but I'm not assuming you're any sort of programmer. In a future post, I'll talk about the same subject, only from the point of software developers.  

## Be Specific

Oftentimes problem reports come in rather vague - "there was a problem with the application." Uhhh, can you be a bit more specific? A lot of the job involves information-gathering, and often it's iterative - you get a problem report, do a little research and analysis, ask for more information, do more analysis, etc.  

## Be Methodical - Work Backwards From the Error

Here's my main point: **Work Backwards**. Start at the point where the error first shows up, figure out what condition(s) cause that to occur, then figure out what causes that, etc. Methodically work your way up the chain of causality. Don't attempt a fix without any analysis, unless you have a really good reason to believe it will work in this situation - in which case, you're not troubleshooting, you're just applying a remedy to a known issue. Good for you for having an effective knowledge base!  

## Don't Make the Problem Worse

Maybe it's because I work with medical software that enhances patient safety, but to me this brings to mind the Hippocratic Oath: "First, do no harm." Don't make the problem worse! I'll say it again: Work backwards. As Gene Kranz in the movie Apollo 13 said, "Work the problem, people." You probably don't have lives at stake, but the same principles apply. Do not jump to conclusions, and don't just "try stuff" hoping it will work! A lot of times, your first thought may be "Hmmm, I wonder if changing the twizzle setting to false might do something." Yes, it might, but it also stands a very good chance of making things worse, or causing a second, unrelated problem.  

## Educate Yourself

The more you know about how the application works, and how the pieces work together, the more effective you'll be. For example, if you know where local data is stored, you can look there to see if things generally look OK (file names, file sizes, etc.). This may lead you to investigate file permissions problems, etc. You don't need to know the nuts and bolts of everything down to the code level, but if you generally know how data flows through the system, what format it's in (XML, JSON, text, binary, etc.), and generally what the data is used for, you'll be much more effective. If you don't know any of this, and it's not written down anywhere, ask a developer to spend a few minutes mapping it all out with you.  

Likewise, you'll make yourself a much more valuable support person if you familiarize yourself with tools that can help in troubleshooting. For example, if you're supporting any kind of web-based application, a tool that allows you to see the HTTP traffic back and forth between the browser and server will be invaluable. (Fiddler is probably the most popular such tool). Learn about it and play with it during your downtime. If you frequently deal with file contention problems in a Windows environment, get familiar with Process Monitor or other free troubleshooting tools from Microsoft.  

## Communicate Effectively

If you need to get help from developers or other people, be sure to summarize the problem with the relevant context. If you just forward an email thread without any additional explanation or context, it will probably not make much sense to the recipient, and they'll have to come back and ask you for more details, which will just waste everyone's time.</div>