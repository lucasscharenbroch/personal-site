---
title: "UW Schedule Converter"
subtitle: "Minimalist webpage for text-to-.ical conversion (JS)"
date: 2024-01-14T20:44:21-06:00
---

### ([Github Link](https://github.com/lucasscharenbroch/uw-schedule-converter))

## The Problem

UW-Madison web portal has a wide variety of pages, some of which[^cse] have UIs that clearly haven't been updated for at least a few years (for some, probably more than a decade).

[^cse]: The main exception to this is [Course Search and Enroll](https://public.enroll.wisc.edu/), which is probably the most heavily trafficked site since enrollment is highly anticipated, frequent, and time-sensitive.

For example, the below is a subpage of the notorious "student center", which has many legacy web-forms.

{{% center-text %}}
<img src="/images/uw-student-center-old-ui.jpg" alt="The UW Student Center: Withdraw From Term"/>
{{% /center-text %}}

There are two primary ways to view a course schedule, and neither way can display the entirety of the schedule's information.

- Way 1: though Course Search and Enroll
    - Looks nice
    - Not much overlapping text
    - The entire schedule easily fits on one screen
    - Almost always fits start and end times
    - Clearly shows the type of each event ("LEC"/"DISC", etc.), and its associated number (which often become important at unexpected times where going through the 2fa login might be annoying)
    - Doesn't show the location of classes

{{% center-text %}}
<img src="/images/cse-schedule.jpg" alt="Course Search and Enroll Schedule"/>
{{% /center-text %}}

- Way 2: through the "Course Schedule App"
    - Doesn't look very nice
    - Lots of overlapping and truncated event-blocks
    - Doesn't fit nicely on one screen, nor at any zoom level
    - Attempts to show all necessary information (course name/number/title, event type/number, location, start/end time), but rarely fits all of it

{{% center-text %}}
<img src="/images/course-schedule-app.jpg" alt="Course Search and Enroll Schedule"/>
{{% /center-text %}}

I want an easy way to obtain a screenshot of my schedule that includes all of the necessary information described above.

My non-automated fix to this was to manually manipulate the html/css of the course-schedule-app schedule so the relevant text fields were visible.
This could be partially automated, using JavaScript to write macros (of sorts) to perform batch operations on each "event" (time block), but it usually ended in some blocks being expanded (and not representing their actual duration), or losing some information (typically the lecture/discussion number).

## The Solution

The core problem here is that it's tricky to prettily display a lot of text in a little box.
This doesn't sound like a problem that's much fun to fix, so I instead decided to find a way of translating the schedule into a universal form (a *.ical* file), which can then be displayed by many different applications (one of which is bound to produce a sufficient image).

At first glance, this seems like a web-scraping problem, but it turns out that the course-schedule-app conveniently produces a uniformly-formatted text-listing of the schedule at the bottom of the print-schedule page.

It looks like this:

```
Monday
LITTRANS 203:  Survey of 19th and 20th Century Russian Literature in Translation I
LEC 001
108 Plant Sciences
11:00 AM to 11:50 AM

COMP SCI 502:  Theory and Practice in Computer Science Education
SEM 003
1263 Computer Sciences
12:05 PM to 12:55 PM

ASTRON 200:  The Physical Universe
LEC 001
1313 Sterling Hall
4:00 PM to 5:15 PM

Tuesday
COMP SCI 559:  Computer Graphics
LEC 001
S413 Chemistry Building
11:00 AM to 12:15 PM

LITTRANS 203:  Survey of 19th and 20th Century Russian Literature in Translation I
DIS 304
119 Noland Hall
1:20 PM to 2:10 PM

... etc.
```

A few regular expressions and some JavaScript magic later, I had a working [webpage](https://lucasscharenbroch.github.io/uw-schedule-converter/).

The iCalendar format was easy to work with; I looked at the [spec](https://datatracker.ietf.org/doc/html/rfc2445) for a few details, which was kind of fun, but it was mostly a matter of injecting the schedule information into boilerplate I harvested from exporting *.ical* files from other applications.

The converter actually provides quite a bit of flexibility over the produced schedule, because it forms the output by evaluating format strings supplied by the user (this is done by calling *eval()* on the user's input enclosed in a [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)).

With this, I was able to achieve my goal.

{{% center-text %}}
<img src="/images/google-calendar-schedule.jpg" alt="Converted schedule displayed in Google Calendar"/>
{{% /center-text %}}
