---
layout: post
title: Ivy and Maven Caching
published: true
tags:
- ivy
- maven
---

The dependency management in Play is a  bit of an afterthought which is being address in V2.0 but for those using V1.X we're stuck with using a Play yaml front-end to Ivy. Most of the time it's absolutely fine but when you are doing local development of non-Play libraries which are stored in a corporate Maven repository (such as Nexus) you end up having problems.

The issue stems from the way in which Ivy (and therefore Play) uses those Maven repositories. Maven repositories are configured in the dependencies.yaml file which supports a subset of the behaviour permitted in the .ivy2/ivysettings.xml configuration file. Once Ivy has resolved the dependency it stores a copy in it's local cache to avoid having to load it from the Maven repository again. And this is the problem. What if the artifact changes in the repository? Generally that's not the case, usually you'd expect a new version of an artifact to be deployed to the repository. That is unless it's a SNAPSHOT version, i.e. an ongoing development version, and this is exactly the situation when you are developing a non-Play library and a Play application at the same time.

Consider the following situation; you are working on a Play application which uses a library that you are also developing, which is not a Play application. You change that library and you perform a Maven "install" to compile and place the snapshot artifact into you local Maven repository. Then you run your Play application to see whether your Play application works using the new version of the library. This isn't going to work because Ivy has cached the library in it's own local repository. It doesn't look for changed artifacts in the Maven repository, and this makes joint development of Play! applications and non-Play libraries a little tricky.

There is a workaround though, instead of defining the Maven repositories through the dependencies.yaml file you define them as a chained resolver in the .ivy2/ivysettings.xml file and there it's possible to define a "changingPattern" on the resolver which indicates that the resolver should re-check the dependency if it's a SNAPSHOT.

Here's how to do this:

<?xml version="1.0" encoding="UTF-8"?>
<ivysettings>
  <settings defaultResolver="Local Ivy Cache"/>

  <resolvers>
    <ibiblio name="Local Maven Repository" m2compatible="true" usepoms="true"
             root="file:///${user.home}/.m2/repository/"/>

    <ibiblio name="Remote Nexus Repository" m2compatible="true" usepoms="true"
             root="http://nexus.remote.url:8080/nexus/content/groups/code/"/>

    <chain name="Local Ivy Cache" changingPattern=".*-SNAPSHOT">
      <resolver ref="Local Maven Repository"/>
      <resolver ref="Remote Nexus Repository"/>
    </chain>
  </resolvers>
</ivysettings>
 
(I'm working on a pure Maven solution for Play so that Ivy isn't involved at all and then this whole problem goes away. It's going to be interesting to see what this process is going to look like when Play v2.X comes along)
