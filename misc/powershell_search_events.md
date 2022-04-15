---
layout: page
title: Search Windows Events using PowerShell
description: desc
---


Sometimes you need to quickly search for Windows Events by some text. Since there may be thousands and thousands of Events dispatched, here's the
quick lookup Powershell script to achieve this goal:


## PowerShell script

    get-winevent application | where { $xml = [xml]$_.toxml() 
    $xml.event.eventdata.data -like '*TEXT_TO_SEARCH*' } | select -first 2000
    

Ok, what do we have here:

First, we get 'Application' type of events and transform it to xml data type style for easier traversal:

    get-winevent application | where { $xml = [xml]$_.toxml()

Next, we are use using 'like' operator and surround it with *TEXT* for inline search:
    $xml.event.eventdata.data -like '*TEXT_TO_SEARCH*' } | select -first 2000

At the end we are restricting Console output to the first 2000 records
    select -first 2000

