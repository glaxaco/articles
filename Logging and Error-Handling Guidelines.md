# Logging and Error-Handling Guidelines

I'm pretty sure I sound kind of logging-obsessed to my fellow developers sometimes. I'm sure I sound like a broken record sometimes when I ask to see the log when troubleshooting an issue, but when logging is done right it provides a clear path toward getting to the root cause of a problem. To my mind, this is about 1000 percent better than the alternative, which mostly consists of guessing and randomly trying things, followed by hiding from management when they come around demanding a status update.  

So, logging is extremely valuable- at least good logging is. But what makes for good logging? I'm tempted to say "I know it when I see it", but in fact over the years I have managed to codify some guidelines about logging and error handling, which I'll lay out here.  

We use [log4net](https://logging.apache.org/log4net/) as our standard logging package for .NET projects. It's been around forever, and I know there are [sexier options](http://serilog.net/) out there, but the guidelines below kind of assume log4net. If you use a different logging framework, I'm pretty sure most of the recommendations will still apply.  

**In a normal production environment, the logging level in configuration should be set to INFO.** In log4net XML configuration, it would look something like this:
```
  <log4net>  
    <appender name="TheAppender" etc.. />  
    <root>  
      <level value="INFO" />  
      <appender-ref ref="TheAppender" />  
    </root>  
  </log4net>  
```

When you are testing or troubleshooting, the level can be changed to DEBUG to gather more information about the system. That means that as you write log statements, you'll need to think about whether you'll want to see them generated as part of normal behavior, or only in debugging/troubleshooting situations.  

**Each class should have its own Logger instance.** We declare it like this:  
```
private static readonly log4net.ILog Log = log4net.LogManager.GetLogger(System.Reflection.MethodBase.GetCurrentMethod().DeclaringType.Name);  
```

This allows the logger to automatically pick up the name of the class, and including the fully-qualified class name on everything allows it to be copied "as is" into a new class in one action without needing to add any "using" directives.  

When coding, **try to anticipate what types of log messages that would be helpful to a future troubleshooter.** This is a bit of a balance, because you want to avoid "spamming" the log with low-value information.  

**Log every request coming in at the highest code level with a level of INFO.** This means controller methods, Main(), etc. This gives you a permanent record of all the requests that you handle.
  
**When logging unexpected error conditions or exceptions you can't recover from, do so with a logging level of ERROR.** This may sound obvious, but if you're consistent about it, you can quickly search through any log for the problem that Support is probably asking you about.
  
**Exceptions that are logged should be done so with the full details of the exception**, including message, stack trace, inner exception (if any), etc. Log4net makes this easy. Having the full stack trace in the log is enormously valuable for troubleshooting.

**Log every condition that can be handled but may be a sign of trouble with a logging level of WARN**. These often provide the "smoking gun" for more inscrutable errors.

**When you get bug reports around unexpected edge cases, log the condition when you're fixing the issue.** These are often WARN statements. 

**Log every database read and write (in Entity Framework, etc.) with a logging level of DEBUG.** These are almost always too frequent to log as INFO (and can leak sensitive information) but are enormously helpful for troubleshooting.

**Log other interactions with the outside world (file interaction, communication with external web services, etc.) with a logging level of INFO**, unless they're so common they really spam the log, in which case go with DEBUG. The same reasoning as for database access - out-of-process calls of any type are a frequent source of failure.

**Log any other helpful data, as needed, with a logging level of DEBUG.** These types of log statements tend to be up to the discretion of the developer, and often they may be removed once a section of code is stable.

**Errors that are not caught anywhere in the code should be caught and logged by the framework if possible.** ELMAH is one tool that does this for ASP.NET web applications. If not possible, the entry point method (controller method, Main()) should have a general catch exception that logs the error.

**If exceptions are caught, they should be re-thrown or completely handled.** Under no circumstances should they leave the data in a bad state.

**Try/catch blocks should be relatively rare and exist to handle specific error conditions.** I've seen a lot of code where developers put a try/catch block in almost every method. This just adds cruft and overhead without adding any value. I think in most cases developers who do this are falling into a "cargo cult" mentality where they think they must add try/catch blocks everywhere in order to "do error handling right." They may be half-right - they should think about error handling in every method, but in my experience most of the time the best thing to do is simply let the exception be thrown and handled up the call stack.

**Do not return status codes to indicate whether there was an error in a method.** If there was an error, throw an exception from the method. This simplifies the calling code significantly as it does not have to check the return values or status codes of method calls. Older programmers who started out their careers using C or other languages without exception support are probably the most guilty of this.

**Sometimes it makes sense to catch exceptions at a lower level in order to log specific details about the exception at that level.** If this is done, be sure to re-throw the exception.  
When re-throwing an exception, it is usually better to use “throw” by itself, rather than throwing the caught exception. Here is a quick example:
```
string base64 = ...
byte[] bytes;  
try  
{  
    bytes = Convert.FromBase64String(base64);  
}  
catch (FormatException ex)  
{  
    // Log the bad data for help in troubleshooting the problem  
    Log.ErrorFormat("Bad base64 string: '{0}'", base64);  
    throw; // this preserves the full stack trace  
    
    // do not do this as it resets the stack trace:  
    // throw ex;  
}
```

In the code above, the only reason there is a try/catch block is so we can log the bad base64 string in the case that it cannot be converted to a byte array. By calling "throw" rather than "throw ex", the stack trace will appear upstream as if it were not caught at all at this level.

**Avoid NullReferenceExceptions as they typically do not contain adequate information about what went wrong.** Validating method arguments and throwing ArumentNullException, ArgumentOutOfRangeException, etc. is the best practice to avoid this.

Along the same lines, **be sure to validate the result of data queries**, including Entity Framework or LINQ queries, and throw an appropriate exception. An exception of this type ("Contact #123 was not found in database") is much more helpful than a NullReferenceException occurring many method calls later.

**Client applications need to be able to handle various failed requests to the server** (server not responding, 404 errors, 500 errors, etc.). If this is not done, it often leaves the client application "hanging" in an unresponsive state.  

Taking a step back, you'll want to think about the process around all this. When a bug or problem is discovered, part of getting it fixed or resolved should be a discussion of what additional logging or instrumentation would have helped to diagnose the issue. Since you generally can't (or shouldn't) do source-level debugging on a production system, you have to rely on production logs, which should help you reproduce the issue in a development environment.