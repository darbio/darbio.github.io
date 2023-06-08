---
layout: post
title: "Uploading a large amount of small files to S3 quickly"
date: 2023-06-08 00:00
comments: false
categories: aws s3 python infrastructure
---

# TL;DR

Make sure you have Python 3 installed and run:

```
# install
pip install boto
pip install python-magic
brew install libmagic // this one got me for a second as I'm running an M1 Mac

# get the script and allow it to be executed
curl https://gist.githubusercontent.com/darbio/8429cb82c00b7d4a2d9069a10f9f08bd/raw/28da5aba3935b05bde3e6d6b96b110b5e8bd3d2b/s3-parallel-put.py --output s3-parallel-put.py
chmod +x ./s3-parallel-put.py

# execute
./s3-parallel-put.py --bucket=BUCKET_NAME ./source-directory
```

# The detail

I recently had a solution which required the generation of a lot (>5 million) small (<50 kb) files which I needed to consume in an iOS application. The files then needed to be served, via https, to the application for consumption in that application. 5 million small(ish) files ended up being around 150 gb. Yikes!

I decided that I would serve these files through AWS S3, using a Cloud Front distribution to cache requests to S3. This isn't the topic of this post, but cue the segue - I decided to do it this way because S3 is an awesome HTTP server, however expensive to use at scale. Requests from Cloud Front to S3 are free, and Cloud Front is significantly cheaper and faster to use for this purpose. I'll discuss this in a future post.

The biggest problem I faced was how to get the 5 million small files up into S3 in the first place. Obviously 150 gb is no small amount to upload, and 5 million is a lot of files to upload manually (or through batches in the S3 UI - which is also sloooooow to upload through).

Enter Python and the [`s3-parallel-put.py`](https://github.com/mishudark/s3-parallel-put) script written by [Tom Payne](https://github.com/twpayne) and maintained by the elusive [misudark](https://github.com/mishudark).

I grabbed the script, installed boto, python-magic and lib magic:

```
pip install boto
pip install python-magic
brew install libmagic // this one got me for a second as I'm running an M1 Mac
```

I ran it and... bang. It doesn't work with Python 3! Which I had installed on my Mac, so I ran it through the Python 2 to 3 script and got something similar that ran, albeit with no output to the console (I could see the files in my AWS S3 bucket).

I needed to get it running qucikly, so I just ripped out all of the logging and replaced it with `print(...)` commands so that I could see what was happening, and voila!

<script src="https://gist.github.com/darbio/8429cb82c00b7d4a2d9069a10f9f08bd.js"></script>

# Performance

I uploaded the 150 gb up to S3, with an average speed of `2233636 bytes/s, 65.0 files/s` (output from the tool), using 31 processes (for some reason, above 31 it bombs out with an exception, so 31 processes it was!). This is on an NBN 50 with 17 Mbps upload.

`./s3-parallel-put.py --bucket=BUCKET_NAME --processes=31 ./source-directory`

NB: I batched the uploads into child directories to help manage the upload, with the output of one of these child directories being `put 27000900141 bytes in 785529 files in 12088.3 seconds (2233636 bytes/s, 65.0 files/s)`.

The Mac activity monitor told me that I was uploading at 2.5 megabytes per second.

![Mac activity monitor](https://github.com/darbio/darbio.github.io/blob/c69a86e7c25e64ea0d76c4b9edb937683069a4e6/_posts/mac_performance_screenshot.png)

Overall, it took around 12 hours to upload the full dataset to S3, which isn't too bad given the amount of data.

# Cost

I'm still waiting for AWS to catch up, but it's going to be a costly (well, for AWS) exercise with the estimate being around $30 USD to perform 5 million `PUT` requests. I'll post a snapshot when AWS has caught up.

Hope this helps!
