---
layout: post
title: Extract Tenable Plugin Downloads URL
tags:
- Tenable
show: true
---
<p>
Stuff
</p>

<!--more-->
<p>
Stuff
</p>

* Do not remove this line (it will not be displayed)
{:toc}

# 1
Log into Tenable Security Center as an admin user. Navigate to System -> Diagnostics File. Leave all the options selected and click "Generate file". Once the file is generated diagnostics file. Extract the archive and open sc-configuration.txt and take note of the 	PluginSubscriptionLogin and PluginSubscriptionPassword fields.

# 2
Constrict the download URL with
  - https://plugins.nessus.org/get.php?u=<PluginSubscriptionLogin>&p=<PluginSubscriptionPassword>&f=SecurityCenterFeed48.tar.gz
  - https://plugins.nessus.org/get.php?u=c2f9f3b7bd9c743c0bf6ea736b39f6e2&p=1253077268824ca78535a490e81120bf&f=all.tar.gz
