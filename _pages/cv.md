---
layout: single
permalink: /cv/
author_profile: true
title: "Curriculum Vitae"

gallery_1:
  - url: /assets/images/cv/sleepsta-1.png
    image_path: /assets/images/cv/sleepsta-1.png
    alt: "Alarm setup screen in Sleepsta"
    title: "Alarm setup screen in Sleepsta"
  - url: /assets/images/cv/sleepsta-2.png
    image_path: /assets/images/cv/sleepsta-2.png
    alt: "Reminder/Volume Setting Screen in Sleepsta"
    title: "Reminder/Volume Setting Screen in Sleepsta"
  - url: /assets/images/cv/sleepsta-3.png
    image_path: /assets/images/cv/sleepsta-3.png
    alt: "Stats Screen in Sleepsta"
    title: "Stats Screen in Sleepsta"

gallery_2:
  - url: /assets/images/cv/risk-assessment-1.png
    image_path: /assets/images/cv/risk-assessment-1.png
    alt: "Build Checker screen in Risk Assessment"
    title: "Build Checker screen in Risk Assessment"
  - url: /assets/images/cv/risk-assessment-2.png
    image_path: /assets/images/cv/risk-assessment-2.png
    alt: "Qualifying products results in Risk Assessment"
    title: "Qualifying products results in Risk Assessment"
  - url: /assets/images/cv/risk-assessment-3.png
    image_path: /assets/images/cv/risk-assessment-3.png
    alt: "Medication Checker screen in Risk Assessment"
    title: "Medication Checker screen in Risk Assessment"

gallery_3:
  - url: /assets/images/cv/study-swipe-1.png
    image_path: /assets/images/cv/study-swipe-1.png
    alt: "Review screen in StudySwipe"
    title: "Review screen in StudySwipe"
  - url: /assets/images/cv/study-swipe-2.png
    image_path: /assets/images/cv/study-swipe-2.png
    alt: "Test taking screen in StudySwipe"
    title: "Test taking screen in StudySwipe"
  - url: /assets/images/cv/study-swipe-3.png
    image_path: /assets/images/cv/study-swipe-3.png
    alt: "Progress tracking screen in StudySwipe"
    title: "Progress tracking screen in StudySwipe"
---

A little over three years ago I left my comfortable career of managing the live production environment for churches to study in the new-at-the-time iOS development program at [Lambda School (now called Bloom Institute of Technology)][lambda]. I spent basically all day every day for nine months reading and writing Swift and learning how to build iOS apps.

I did well enough that after ten weeks they asked me to pause my studies for a little bit and act as the class lead for a new cohort of students. So for ten weeks in the middle of my time at Lambda, I was the class lead for the iOS4 cohort, helping students who needed help, reviewing/debugging their code, and presenting code challenges.

Ten days after finishing that course, I started a job at [Madwire][madwire] working on their [Top Rated Local][trl] app. We used a modified version of the [VIPER architecture][clean-swift], shot for 100% code coverage with our unit tests and generated all our UI programmatically so I got a crash course in all three of those things there.

After that I joined [Ibotta][ibotta], working on their [iOS app][ibotta-app] as a part of the Radicial Simplification team. Our goal was to reduce friction as much as possible for the users of the app by showing them (only) content they care about and removing unnecessary roadblocks in the process of earning rewards. I got to work with some incredible people and learned a lot about iOS architecture and working cross-functionally in a large organization while I was there.

Most recently I accepted a position as Lead iOS Developer at [Hallow][hallow], a prayer and meditation app. At Hallow I am responsible for the iOS app, maintaining the code base, establishing best practice and releasing new features and bug fixes on a regular cadence.


I have also been [a photographer/videographer in my free time](http://light-and-lens.com) for the last several years. I’ve noticed there is a surprising amount of overlap between doing those things and software development. You have to be able to read documentation. You have to be able to learn stuff for yourself. You have to figure out how to solve problems in creative (and sustainable, and repeatable)  ways. And you have to think about the user’s experience every step of the way. I enjoy applying those skills to the work I do now, and I'm sure they'll serve me well for whatever comes next!

-----
*...Dillon has a clear attention to detail, which is a trait that is very hard to find (or train). He thinks through problems patiently, has a knack for noticing edge cases, and also seems like an amiable and friendly person to work with. He was able to answer every theoretical question and solve every problem I asked him to solve without any issue…*  
{: .text-center}
– Antonio Tari, [Skilled Interview](https://www.skilledinc.com/report-card/2103)
{: .text-right}

## Projects
### StudySwipe
![image-left](/assets/images/cv/study-swipe-icon.png){: .align-left}
[StudySwipe](https://apps.apple.com/us/app/studyswipe/id1470980976) is a flash-card style app that [Ben Hakes](https://twitter.com/benhakes) and I built as a part of Lambda's 2019 Summer Hackathon. We wanted an app that would help us prepare for technical interviews, which we were both in the middle of going through at the time. We built the bulk of the functionality in the 48 hour period of the hackathon and we ended up winning "Best Native App". We have since continued to work on it in our spare time and recently released it on the App Store. It currently contains about 150 questions that are relevant for iOS Development-related technical interviews, but we hope to expand to other subjects soon.

**Primary Frameworks:**
CoreAnimation, CoreData, Programmatic UI, Down (for markdown rendering), Vapor (soon)
{% include gallery id="gallery_3" %}

### Sleepsta
![image-left](/assets/images/cv/sleepsta-icon.png){: .align-left}
[Sleepsta](https://sleepsta.netlify.com/) is a sleep tracking/alarm app. The iOS app tracks your motion throughout the night to calculate a sleep quality and lets you view your daily stats on a graph. I built it as my capstone project at Lambda School, over the course of five weeks. I was the only iOS dev on a team with 5 web devs. You can see [the code on GitHub](https://github.com/labs11-sleep-track/labs11-sleepTrack-iOS), or [read the blog posts I wrote about the experience](https://dillon-mce.com/tags/#labs).

**Primary frameworks:**
CoreMotion, UserNotifications, AVFoundation, MusicKit, Google Sign-In
{% include gallery id="gallery_1" %}

### Life Insurance Risk assessment
![image-left](/assets/images/cv/risk-assessment-icon.png){: .align-left}
For Lambda’s 2019 Winter Hackathon, I was a part of a team that built an app for life insurance agents over 30 hours. We built an app that drastically simplifies the tables that agents have to check against to see if a person qualifies for a particular plan. There are two main features. The "build checker", which lets you put in some basic information about a person, and get back a list of plans they aren't disqualified from. And a "medication checker", which lets you see if a certain medication disqualifies you from a given plan. You can see [the code on Github](https://github.com/dillon-mce/winter-hackathon-2019/tree/master/Risk%20Assessment), or [read the blog post I wrote about it]({% post_url 2019-01-08-Lambda-2019-Winter-Hackathon %}).

**Primary frameworks:**
CoreAnimation, UIKit
{% include gallery id="gallery_2" %}

## Previous Employers
**[Hallow][hallow] - Lead iOS Engineer | 06/21 - Present**
Hallow (pronounced like aloe) is a prayer and meditation app, focused primarily on audio experiences that help promote spiritual, mental and physical growth. My primary responsibility is to add features to the iOS app and to squash bugs that pop up along the way.

**[Ibotta][ibotta] – Senior Mobile Engineer | 11/19 - 06/21**  
Ibotta is a Denver based mobile technology company that strives to "make every purchase rewarding" by enabling its users to earn cash back on purchases in a variety of ways. I work as an iOS engineer on the Radical Simplification team, focused on making the [iOS app][ibotta-app] simpler and easier to use.

**[Madwire][madwire] – Mobile Software Engineer | 07/19 - 11/19**  
Madwire is a Marketing and Design company (i.e. M-A-D) that offers a whole suite of digital marketing software and professional marketing services through a platform called Marketing 360. I work on the mobile version of their Yelp-like product called [Top Rated Local][trl] fixing bugs and adding features through test driven development with the VIPER pattern.

**[Lambda School][lambda] - iOS4 Class Lead | 11/18 - 02/19**  
Lambda School is an online software development school offering tracks in full-stack web, iOS, or Android development as well as Data Science and UX Design. I tracked student attendance and reviewed students' code daily. I also wrote and presented code challenges and presented guided projects for students who needed to repeat material.

**[Rocky Mountain](https://rocky.church/) - Technical Director | 05/15 - 09/18**  
Rocky Mountain Christian Church is a community-focused church with around 2500 members and six services per week on two campuses. I oversaw audio, video and lighting systems, and managed regular live- streaming of sermons between campuses. I also coordinated volunteers and generally helped solve any non-IT technical problems.

## Education:
**[Lambda School][lambda] - iOS Development | 08/18 - 06/19**  
I studied iOS app development at Lambda, primarily with Swift and the main Cocoa frameworks, following a general MVC pattern. I also studied general Computer Science and got some exposure to Objective-C, Python, and C.

**[Ozark Christian College](https://occ.edu/) - B.A. Christian Ministry | 08/09 - 05/14**  
I primarily studied ancient Greek and Hebrew at Ozark, along with general psychology and counseling, philosophy and theology. I read tens of thousands of pages and wrote hundreds.

[ibotta]: https://home.ibotta.com/
[ibotta-app]: https://apps.apple.com/us/app/ibotta/id559887125
[madwire]: https://www.madwire.com/
[lambda]: https://lambdaschool.com/
[clean-swift]: https://clean-swift.com/
[trl]: https://apps.apple.com/us/app/top-rated-local/id1270356201?uo=4
[hallow]: https://hallow.com
