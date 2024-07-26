---
layout: post
title: Beautiful log test on Gradle
tags: [GradlePlugin, Log]
comments: true
---
When executing tests in a Gradle project, it is very helpful to see detailed information from the test logs, such as the package name, class, test name, result, and execution time like this:

<pre style="color: #000000; background-color: #333333; padding: 10px; border-radius: 5px;">
<code>
> Task :test

  com.thachlp.cdc.consumer.querybuilder.<span style="color: #1087dd;">QueryBuilderDeleteTest</span>

    ✔ <span style="color: #008000;">testBuild_DeleteQuery()</span> (867ms)

  com.thachlp.cdc.consumer.querybuilder.<span style="color: #1087dd;">QueryBuilderInsertTest</span>

    ✔ <span style="color: #008000;">testBuild_InsertQuery()</span>

  com.thachlp.cdc.consumer.querybuilder.<span style="color: #1087dd;">QueryBuilderUpdateTest</span>

    ✔ <span style="color: #008000;">testBuild_UpdateQuery()</span>

  <span style="color: #008000;">3 passing</span> (2s)
</code>
</pre>

Setting up the `test-logger` plugin is very simple. Just add the plugin and configuration to your **`build.gradle`** file:

```groovy
    plugins {
        // Check the compatibility with gradle version
        id "com.adarshr.test-logger" version "4.0.0"
    }

    testlogger {
        theme 'mocha'
        slowThreshold 500
    }
```
