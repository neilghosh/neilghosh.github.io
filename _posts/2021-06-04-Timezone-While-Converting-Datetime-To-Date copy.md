---
layout: post
title: "Timezone Conversion Java"
description: "Its becomes tricky when you are converting a timestamp to a date, especially between java.util.Date and say java.time.LocalDate"
---

The traditionally used `Java.util.Date` represents a specific moment of time like now(), or the moment I was born.
It doesn't have any timezone it's just a point in the arrow of time. People may want to translate it to their convenient timezone i.e. if it's an evening at 7 PM in IST, it is also 1.30 PM in UTC. The moment of time does not change.
      
The problem is when someone gives you a moment of time and asks you what date it is. By saying date the person is expecting something like "1st Jan 20202" or 2020-01-01 and don't care about time. If he would have given me "1st Jan 2020 evening 7 pm", I would have told him its "1st Jan 2020". 
      
It's probably true in most cases because we know the person probably cares only about his timezone. So no matter which timezone he is in the answer to the question is still "1st Jan 2020". However, we can't confidently say that. If the person is telling the time (and date) in his timezone say (UTC) and I am answering it to someone else who is in IST for the person I am answering to it's already 2nd Jan because IST is 5.30 hours ahead of UTC. So in a software system, the timezone of the person who is inputting time is not necessarily the same as the other people using it.
      
So when we convert a point of time to just a Date value we need to consider the date at what timezone. The same is the case when data is saved in `java.util.Date` which is a point of time with no timezone information. While displaying the date it could be different for the different user based on their timezone.

```
        print(LocalDate.of(2021, 1, 1), toDate(OffsetDateTime.of(2021, 1, 1, 23, 0, 0, 0, UTC), "Etc/UTC"),
                "Convert UTC to UTC");
        // If the time mentioned is 11pm at night at London, the date at Kolkata is next day.
        print(LocalDate.of(2021, 1, 2), toDate(OffsetDateTime.of(2021, 1, 1, 23, 0, 0, 0, UTC), "Asia/Kolkata"),
                "Convert UTC to IST");
        print(LocalDate.of(2021, 1, 1),
                toDate(OffsetDateTime.of(2021, 1, 1, 23, 0, 0, 0, ZoneOffset.of("+0530")), "Etc/UTC"),
                "Convert IST to UTC");
```
Output
```
Expected : 2021-01-01 Actual : 2021-01-01 - Convert UTC to UTC
Expected : 2021-01-02 Actual : 2021-01-02 - Convert UTC to IST
Expected : 2021-01-01 Actual : 2021-01-01 - Convert IST to UTC
```
