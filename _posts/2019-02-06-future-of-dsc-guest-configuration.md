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
Let's be clear, don't expect a new version of WMF coming, just security patches. Afterall, PowerShell 6 (aka PS Core or pwsh) went a long way to be decoupled from WMF and have a life of its own, allowing a much higher release cadence and open-source development.

It's worth noting that the DSC DSL is still available in PowerShell 6...

The DSC Pull & Report Server was not really intented to be offered as a possible Configuration Management solution, but was offered as a **feature in Windows Server** so we could experience the Pull Mode, and start contributing and using DSC Resources.
I think that experience with the Pull Server was both what attracted people to implement DSC in their datacentre (it is 'free', minus the Windows license), and what frustrated most who implemented it early on in production (or wanted to).

Against all odds, the last version of Windows Server was released with the much sought after feature to use a SQL Server backend to the Pull server. That said, it has been made clear that would be the last Pull Server microsoft would release (which means the feature will disappear in further releases). For a long time Microsoft has said that Azure Automation was the recommended approach for implementing the DSC Pull Model.




Azure Automation State Configuration

AWS DSC


> <h3>DSC is NOT Dead, long live Guest Configuration!</h3>