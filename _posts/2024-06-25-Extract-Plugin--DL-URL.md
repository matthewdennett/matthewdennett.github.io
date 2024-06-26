---
layout: post
title: Tenable Plugin Downloads
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
