---
title: "Software Maintenance at Commit-Time"
abstract: "The maintenance and evolution of complex software systems account for more than 70% software’s life cycle. Hundreds of papers have been published with the aim to improve our knowledge of these processes in terms of issue triaging, issue prediction, duplicate issue detection, issue reproduction and co-changes prediction. All these publications gave meaning to the millions of issues that can be found in open source issue & project and revision management systems. Context-aware IDE and think tank in open source architecture open the path to approaches that support developers during their programming sessions by leveraging past indexed knowledge and past architectures. In this report, we present four approaches: BUMPER, JCHARMING, PRECINCT, BIANCA. When combined these approaches (i) provide the possibility to search related software artifacts using natural language, (ii) accurately reproduce field-crash in lab environment, (iii) prevent the introduction of clones / issues at commit time."
bibliography: config/library.bib
classoption: conference
author: 
- name: Mathieu Nayrolles
  affiliation: SBA Lab, ECE Dept, Concordia University
  location: Montréal, QC, Canada
  email: mathieu.nayrolles\@concordia.ca
---

# Introduction


Software maintenance activities such as debugging and feature
enhancement are known to be challenging and costly [@Pressman2005].
Studies have shown that the cost of software maintenance can reach up to
70% of the overall cost of the software development life cycle
[@HealthSocial2002]. Much of this is attributable to several factors
including the increase in software complexity, the lack of traceability
between the various artifacts of the software development process, the
lack of proper documentation, and the unavailability of the original
developers of the systems.

More than three decades of research in different fields such mining software repository, default prevention, clone detection, program comprehension or software maintenance aimed to understand better the needs of and challenges faced by developers when entertaining a program.
Researchers have proposed hundreds of approaches and tools to improve the maintenance processes.
Yet, the large adoption of these tools by the practitioners remains limited [@Lewis2013; @Foss2015; @Layman2007; @Ayewah2007; @Johnson2013].
Factors that prevent such an adoption are still an open question.
Nevertheless, based on developers interviews, several hypotheses have been formulated.
For example, developers are known to have trust issues with statistical models based on process or code metrics.
Another hypothesis for the limited adoption of software maintenance approaches is the lack of integration with developers' workflow.
Finally, developers also express concerns regarding the numbers of warnings, the general heaviness of information provided by software maintenance tools and the lack of clear corrective actions to fix a given warning.

In this thesis, we propose to address some of the issues mentioned above by focusing on developing techniques and tools that support software maintainers at commit-time. 
As part of the developer’s workflow, a commit marks the end of a given task or subtask as the developer is ready to version the source code. 
Commits are bite-sized units of work that are potentially ready to be shared with the rest of the organization [@OSullivan2009].

We propose a set of approaches in which we intercept the commits and analyze them with the objective of preventing unwanted modifications to the system. By doing so, we do not only propose solutions that integrate well with the developer’s work
flow, but also there is no need for software developers to use any other
external tools. 
More precisely, we propose the following contributions: (a) an aggregated bug report/bug fix repository system, (b) a clone prevention technique at commit-time, (c) a risky change detection technique at commit-time and, (d) a bug reproduction technique based on directed model checking and crash traces.

In the rest of this paper is organized as follows: In section \ref{sec:prel} we describe the preliminary works we conducted to support our contributions.
Then, in section \ref{sec:approach} we present the way we plan to lower the resistance to the adoption of software maintenance tools by developers.
Finally, in section \ref{contributions} we summarize our contributions.

# Preliminary Work {#sec:prel}	

Our preliminary work aimed to understand the rationale behind the low adoption of software maintenance tools by practitioners.
To do so, we conducted a systematic literature review of several research fields.
Indeed, to understand the problem, we had to survey not only the literature related to software maintenance but also the literature related to developers themselves: How developers work? How developers thinks? What are the thoughts processes and mental models behind software maintenance activities?

In this section, we report the findings that led us to believe that, while current method and tools are efficient to improve the software maintenance activities, they do not interact with the developers at the right time in the development process and lack some key features.

First of all, there is a lack of integration of developers workflow of most approaches ([ @Ayewah2008b; @Findbugs2015] are some notable examples).
If developers want to improve their processes using these approaches, they will have to download, install and understand them to achieve a given task.
Having an external tool to perform software maintenance or creation activities do not fit into the workflow of developers (i.e., coding, testing, debugging, committing). 
Moreover, tools are specialized for a given tasks (i.e., feature location with a command line tool, development and testing code with an IDE, development, and testing front end code with another IDE and a browser, etc.).
Using them lead to context and workspace switching which can hinder the productivity of developers [@Robertson2006; @Beckwith2006].

To avoid context and workspace switching researchers have heavily resorted to IDE plugins.
IDE plugins are extensions to developers IDE that performs specialized tasks and reports its findings directly into the IDE.
Unfortunately, developers report that these IDE plugins do not provide the corrective actions that would circumvent a given warning.
Take, for example, FindBugs [@Hovemeyer2004], a popular bug detection tool. This tool detects hundreds of bug signatures and reports them using an abbreviated code such as `CO_COMPARETO_INCORRECT_FLOATING`. 
Using this code, developers can browse the FindBug’s dictionary and find the corresponding definition _“This method compares double or float values using pattern like this: $val1 > val2~?~1 : val1 < val2~?~-1 : 0$”_.
While the detection of this bug pattern is accurate, the tool does not propose any corrective actions to the developers that can help them fix the problem.
Moreover, it has been reported in the literature that the output of existing maintenance tools tends to be verbose at the point where developers decide to simply ignore them [@Arai2014;  @Kim2007c;  @Shen2011].
Another example, related to clone detection, found that they are six different reasons that trigger the use of clones (e.g., copy and paste of code examples,  a reimplementation of the same functionality in a different language, etc. ). 
Developers are aware that they are creating clones in five out of six situations. 
In such cases, warnings provided by IDE plugins will only be disturbing for developers.

Finally, developers perceive statistical models aiming to improve the maintenance processes as untrustworthy black-boxes.
In risky changes detection, where approaches warn developers about changes to the software that are likely to introduce a new bug, statistical model are frequently based on code metrics (i.e. number of line added/deleted, numbers of files/package modified, etc).
Then a statistical model is built with the changes that are known to have introduced bugs in the software, and if a new change is over the fit, a flag is raised.
Developers do not trust statistical model mainly because they do not provide examples but are only based on model fit.

According to these different findings, we believe that software maintenance tools and approaches should interfaces with developers at commit-time and, in addition, provide corrective actions for each warning.
Interacting with software developers at commit-time avoid to resort to external tools that induce context switching or IDE plugins that can be distracting.
Also, software maintenance at commit-time has the advantage to intervene before the code actually reaches the central repository and become pullable by the other members of the organizations.
Indeed, approaches consisting in monitoring central code repository for quality check exist.
These approaches provide email reports to developers.
We argue that this is not practical because clones, defects, and mistakes can be synchronized by other team members, which may lead to challenging merges if corrective actions are undertaken.

Also, it is our opinion that the _right_ time for the "Just-In-Time Quality Assurance" [@Kamei2013b] movement---where defect prediction models identify risky changes as they get committed---is commit-time. 

# Research Approach {#sec:approach}

Our preliminary works have been used to conceptualize tools on approaches that could fit into commit-time software maintenance.
The approaches we proposed do not hinder the productivity of developers by interrupting them yet, they integrate themselves seamlessly into the day-to-day workflow of developers.

To create approaches that are perceived as trustworthy by developers, we need to provide, in the case of risky changes preventions, examples on why the change is considered risky and present corrective actions to the developers.
To do, so we propose BUMPER (An Aggregate Bug-Fix Repository for Developers and Researchers) in section \ref{sec:bumper} in combination with PRECINCT and BIANCA presented in section \ref{sec:ct} both.
Finally, if despite commit-time maintenance, defects reach the central repository and thereafter release to production, it will lead to bug reports.
One of the most crucial piece of information to fix a bug is the step to reproduce [@Bettenburg2008].
Consequently, we propose an approach named JCHARMING to reproduce the bug.


## An Aggregate Bug-Fix Repository for Developers and Researchers {#sec:bumper}

In this work, we introduce BUMPER (BUg Metarepository for dEvelopers and Researchers), a web-based infrastructure that can be used by software developers and researchers to access data from diverse repositories using natural language queries in a transparent manner, regardless of where the data was created and hosted [@Nayrolles2016b].
The idea behind BUMPER is that it can connect to any bug tracking and version control systems and download the data into a single database.
We created a common schema that represents data, stored in various bug tracking and version control systems.
BUMPER uses a web-based interface to allow users to search the aggregated database by expressing queries through a single point of access.
This way, users can focus on the analysis itself and not on the way the data is represented or located.
BUMPER supports many features including: (1) the ability to use multiple bug tracking and control version systems, (2) the ability to search very efficiently large data repositories using both natural language and a specialized query language, (3) the mapping between the bug reports and the fixes, and (4) the ability to export the search results in JSON, CSV and XML formats.

Importantly, BUMPER differs from other approaches such as Boa [@Dyer2013] because (a) it updates itself every day with the new closed reports, (b) it proposes a clear and concise JSON API that anyone can use to support their approaches or tools.


## An Approach for Preventing Risky Changes and Clone Insertion at Commit-Time {#sec:ct}

Code clones appear when developers reuse code with little to no modification to the original code. Studies have shown that clones can account for about 7% to 50% of the code in a given software system [@Baker; @StephaneDucasse].
Nevertheless, clones are considered a bad practice in software development since they can introduce new bugs in code [@Juergens2009].
If a bug is discovered in one segment of the code that has been copied and pasted several times, then the developers will have to remember the places where this segment has been reused to fix the bug in each place.

In this research, we present PRECINCT (PREventing Clones INsertion at Commit-Time) that focuses on preventing the insertion of clones at commit time, i.e., before they reach the central code repository. 
PRECINCT is an online clone detection technique that relies on the use of pre-commit hooks capabilities of modern source code version control systems. 
A pre-commit hook is a process that one can implement to receive the latest modification to the source code done by a given developer just before the code reaches the central repository. 
PRECINCT intercepts this modification and analyses its content to see whether a suspicious clone has been introduced or not. 
A flag is raised if a code fragment is suspected to be a clone of an existing code segment. In fact, PRECINCT, itself, can be seen as a pre-commit hook that detects clones that might have been inserted in the latest changes with regard to the rest of the source code.

Similar to clone detection, we propose an approach for preventing the introduction of bugs at commit-time. 
Many tools exist to prevent a developer to ship *bad* code [@Hovemeyer2007]. 
However, these tools rely on metrics and rules to statically and/or dynamically identify sub-optimum code. 
Our approach, called BIANCA (Bug Insertion ANticipation by Clone Analysis at commit-time), is different than the approaches presented in the literature because it mines and analyses the change patterns in commits and matches them against past commits known to have introduced a defect in the code (or that have just been replaced by better implementation).


## A Bug Reproduction Technique Based on a Combination of Crash Traces and Model Checking

When preventing measures have failed, and bugs have to be fixed; the first step is to reproduce what happened on sites.
Crash reproduction is an expensive task because the
data provided by end users is often scarce [@Chen2013]. 
It is, therefore, important to invest in techniques and tools for automatic bug reproduction to ease the maintenance process and accelerate the rate of bug fixes and patches. 
Existing techniques can be divided into two categories: (a) On-field record and in-house replay [ @Roehm2015], and (b) In-house crash explanation [@Zuddas2014].

We propose an approach, called JCHARMING (Java CrasH Automatic Reproduction by directed Model checkING) that uses a combination of crash traces and model checking to reproduce bugs that caused field failures automatically [@Nayrolles2015; @Nayrolles2016d]. 
Unlike existing techniques, JCHARMING does not require instrumentation of the code. 
It does not need access to the content of the heap either. 
Instead, JCHARMING uses a list of functions output when an uncaught exception in Java occurs (i.e., the
crash trace) to guide a model checking engine to uncover the statements that caused the crash.
Such outputs are often found in bug reports.

# Contributions {#contributions}

Based on our preliminary works, we conceptualized four approaches (BUMPER, PRECINCT, BIANCA and JCHARMING) that allows the aggregation of bug-fix, the prevention of clone insertion and risky and changes before they reach the central repository and the reproduction of bug, namely.

Our approaches work at commit-time and are efficient trade-offs between external tools, IDE-plugin and remote approaches for clone detection, risky changes detection, and bug reproduction and, as such, we believe that it addresses major factors that contribute to the slow adoption of software maintenance approaches by practitioners. 

We believe that software maintenance at commit-time is a non-intrusive yet effective way to improve software quality altogether as it will combine state-of-the-art approaches in clone detection, risky change detection and bug reproduction without the context switching or distraction they provoke according to developers.
Indeed, a commit marks the end of a particular programming task which is consequent enough to be shared with the organization.
The task at hand being completed there is no risk of interrupting the thoughts processes of developers and hindering their productivity. 
In addition, we argue that intervening at commit-time is better than, for example, remote approaches that analyse the quality of the code after they reach the central repository and send email reports because mistakes can be pulled by other members of the organization further complexifying their removal.
Finally, emails report comes in an asynchronous manner and developers, in case of warnings, have to reconstruct the mental models they were in for the suspected tasks.

\section*{References}
\setlength{\parindent}{0pt}
