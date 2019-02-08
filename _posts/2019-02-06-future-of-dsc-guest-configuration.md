---
ID: 4
post_title: The Future of DSC - Introducing Azure Guest Configuration
author: gaelcolas
post_excerpt: ""
layout: post
permalink: https://synedgy.com/future-of-dsc-guest-configuration
published: false
post_date: 2019-02-18 07:30:00
---

<h1>The Future of DSC - Introducing Azure Guest Configuration</h1>

Across the community and its different channels, I found many people struggling to decrypt Microsoft's position around
Desired State Configuration (on-prem or Azure), its future, and Configuration Management on Windows.
As a Configuration Management specialist, I make a point to follow and analyse this area closely, and will share my thinking below.

<h2>Some Context</h2>

Although I recommend getting [The DSC Book](https://leanpub.com/the-dsc-book) by [Don Jones](https://twitter.com/concentrateddon) and [Melissa Januszko](https://twitter.com/thedevopsdiva) to quickly learn all you need to know how to be proficient with DSC, or [Ravikanth Chaganti](https://twitter.com/ravikanth)'s [Pro PowerShell Desired State Configuration](https://www.apress.com/us/book/9781484234822) for going further, here's just enough context to understand how things are evolving.

The first version of DSC was released as part of the Windows Management Framework 4 (WMF4) in 2013, then had a couple of
minor updates via Windows Updates, before another version coming with WMF 5 and properly released (fixed) in 5.1 (2016), introducing
some new features.

The DSC components, at a high level are:
1. the Domain Specific Language (DSL) & Commands
2. the Local Configuration Manager (one for Windows & one for Linux)
3. the Pull Server & the Report server
4. the DSC resources

And this is what they do:
1. The DSL is what allows us to use the Declarative Syntax within PowerShell to define the configuration policy of your systems, and compiles into MOF Documents.
2. The LCM is the agent what enacts those MOF documents, whether on demand or periodically when using the Pull model.
3. The pull server is a glorified File server that implements the [MS-DSCPM protocol](https://msdn.microsoft.com/library/dn393548.aspx) to talk to the LCM.
    The Report server is a data collector aggregating the results of each DSC runs in its database.
4. The DSC resources are some sorts of interfaces enabling idempotent characteristics to scripts configuring atomic resources, or a composition of different resources.
The most popular are written for the Windows LCM supporting PowerShell 5.1, but some exists for Linux and can be written in [Python or pretty much any language](https://www.powershellmagazine.com/2015/02/26/working-with-powershell-dsc-for-linux-part-4/).


<h2>Tightly coupled legacy</h2>

What needs the more improvements is the LCM, but the current version was released as part of WMF 5.1 for Windows, and a whole different code base for Linux. It was also built atop WMI, hence the MOF format used for compiled configurations.
Let's be clear, don't expect a new version of WMF coming, only security patches. Afterall, PowerShell 6 (aka PS Core or pwsh) went a long way to be decoupled from WMF and have a life of its own, allowing a much higher release cadence and open-source development.

It's worth noting that the DSC DSL is still available in PowerShell 6...

The DSC Pull & Report Server was not really intented to be offered as a possible Configuration Management solution, but was offered as a **feature in Windows Server** so we could experience the Pull Mode, and start contributing and using DSC Resources.
I think that experience with the Pull Server was both what attracted people to implement DSC in their datacentre (it is a 'free' PowerShell Configuration Management Tool, minus the Windows license), and what frustrated most who implemented it early on in production (or wanted to, it's not a feature-rich 'Solution').

Against all odds, the last version of Windows Server was released with the much sought after feature to the Pull server: using a SQL Server backend. That said, it has been made clear that would be the last Pull Server microsoft would include in Windows (which means the feature will disappear in further releases). For a long time Microsoft has said that Azure Automation was the recommended approach for implementing the DSC Pull Model.

The DSC Resources have been doing well and are still actively developped by the growing community, some are looked after by Microsoft (DSC Resource Kit), and used on Windows by the users of many Configuration Management tools: Chef, Puppet, Ansible and of course DSC, but also in Azure with DSC Extension and Azure Automation DSC.

<h2>Intent and realignment of the DSC Team</h2>

Although it was quite clear that DSC needed to be decoupled from WMF as soon as PowerShell Core's project started even before becoming public, DSC's development did not stop. However, it did not get the same publicity and openness as PowerShell Core. [DSC for linux](https://github.com/Microsoft/PowerShell-DSC-for-Linux) already existed, but the tight dependency of the Windows DSC LCM with WMI means there is two code base and a lot of overhead (i.e. OMI) to be compatible cross platform and with the legacy. Not to mention the MOF file format.
If DSC was to innovate, it needed to part from this old model, like PowerShell Core did.

The DSC team, then led by Mark Gray, started working on a new DSC LCM - hastly coined _DSC Core_ - and published the first DSC Planning update blog to explain the work in progress, challenges, and Philosophy.

The intent, from then and confirmed by new Program Manager, Michael Greene at the PS Summit and PSConfEU 2018, was to release the new version of DSC. Possibly Open-Sourcing it as early as they would be capable of, despite the increased effort required when developping in the open, as they learnt from PS Core.

But in June, the almost-traditional re-org month at Microsoft, their effort had been realigned with the company's goal, which is both good and bad from a DSC User perspective, especially on-prem users.
I'd refer you to my previous post about [Understanding the new Microsoft](https://gaelcolas.com/2018/06/04/for-it-pros-understanding-the-new-microsoft/) to explain the motives and patterns, but in short it's "Cloud First" with and agile development that continuously deliver for fast feedback.

What that means for the work on the new native LCM is to develop and use it actively for solving existing Azure challenges - some sort of eat your own dog food approach.
It has the benefit to be:
- far less _theoretical_ than building a generic framework
- support an untethered release and feedback cadence
- enable iterative solution development that uses the components as they're developed, and develop as they need
- Access to millions of machines, telemetry, and huge scale to solve modern use cases

So the sad part is, it's no longer the new DSC that we've been promised for a while and there won't be a tool you can use




<h2>Azure Automation State Configuration</h2>

<h2>AWS DSC</h2>


> <h3>DSC is NOT Dead, long live Guest Configuration!</h3>

DSC Planning updates:
- September 2017: [DSC Future Direction Update](https://blogs.msdn.microsoft.com/powershell/2017/09/12/dsc-future-direction-update/)
- January 2018: [Desired State Configuration (DSC) Planning Update - January 2018](https://blogs.msdn.microsoft.com/powershell/2018/01/26/dsc-planning-update-january-2018/)
- September 2018: [Desired State Configuration (DSC) Planning Update – September 2018](https://blogs.msdn.microsoft.com/powershell/2018/09/13/desired-state-configuration-dsc-planning-update-september-2018/)
