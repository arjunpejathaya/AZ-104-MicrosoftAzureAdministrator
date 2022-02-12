---
title: Online Hosted Instructions
permalink: index.html
layout: home
---

# Content Directory

Required labs files can be DOWNLOADED FROM [HERE](https://github.com/arjunpejathaya/AZ-104-MicrosoftAzureAdministrator/archive/master.zip)

Hyperlinks to each of the lab exercises are listed below.

The following instructions are for performing labs alongside Virtual Instructor Led Classes.

The labs can be completed with the $100 Azure Pass issued for attendees of the course.

It is recommended to try these labs on a Personal Owned Machine and not on Corporate Assets.

## Labs

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Module | Lab |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}


