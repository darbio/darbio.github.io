---
layout: post
title: "OAuth2 authentication with WebAPI, using Windows Server 2012 R2 and ADFS"
date: 2014-06-25 00:00
comments: true
categories: 
---

This post will walk through how to set up a .Net WebAPI 2 application to use and accept bearer tokens from ADSF OAuth 2.0. The WebAPI is also set up to allow local logins, and will be extensible to allow logins using other OAuth 2.0 providers such as Facebook, Google and Twitter.

