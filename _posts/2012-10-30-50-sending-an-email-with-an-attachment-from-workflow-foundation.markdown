---
layout: post
title: "Sending an email with an attachment from Workflow Foundation"
date: 2012-02-24 23:32
comments: true
categories: 
---
The problem:
----------

The `SendEmailActivity` in WF sucks... because you can't send an email with attachments from it.

I want to be able to send an email from a SharePoint WorkFlow, using the SP Outbound Mail Service and adding some attachments to the message.

The workaround:
---------

Use the .Net `MailMessage` class (`System.Net.Mail`) and a Code activity to send the email:

Non-SharePoint example:

<script src="https://gist.github.com/darbio/1cdbf2700013e37e530e.js?file=normal.cs"></script>

SharePoint example:

<script src="https://gist.github.com/darbio/1cdbf2700013e37e530e.js?file=sharepoint.cs"></script>

SharePoint Gotcha's
------------

SharePoint's smtp server must be configured in Central Admin, otherwise you will get a `NullReferenceException` at `web.Site.WebApplication.OutboundMailServiceInstance`

Restart the SharePoint Timer Service to refresh it's assembly cache if the workflow does not kick in

`SPContext` is not available in the Timer Job Context (under which the WF will run)

Get the source code here
