---
layout: post
title:  "[Unity]Unity中Time.timeScale详解"
date:   2017-02-25 01:15:10
categories: 01---【Unity】##########
comments: true
---

Unity中Time.timeScale详解

1、timeScale不影响Update和LateUpdate，会影响FixedUpdate。

2、timeScale不影响Time.realtimeSinceStartup，会影响Time.timeSinceLevelLoad和Time.time。

3、timeScale不影响Time.fixedDeltaTime和Time.unscaleDeltaTime，会影响Time.deltaTime。