---
layout: post
title: Exception handling in Scala
---
# Exception handling in Scala

## `None` is better than `null`

Use this to e.g. casting a nullable value to string and get `None` in case of `null`
```scala
import scala.util.control.Exception.catching
def getSomeOrNone[OUT](out: => OUT): Option[OUT] = catching(classOf[Throwable]).opt(out)
```
