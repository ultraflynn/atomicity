---
layout: post
title: Jackson Serialization
published: true
tags:
- jackson
---

Jackson JSON provides many different ways to customize the serialization process of a class, most of these methods utilise annotations on the classes themselves. I recently had the requirement to customize the serialization of a class when I had no access to change the source code of that class. That meant that using annotations wasn't an option and I had to find another way.

After a fair amount of googling I stumbled across the following page (written by the author of Jackson):

Every day Jackson usage, part 3: Filtering properties

There are 7 options detailed in this post, 6 of which use annotations on the class itself. Option 7 is my only option then. Here it is:

7 . Most extreme way to filter out properties: BeanSerializerModifier

And if ability to define custom filters is not enough, the ultimate in configurability is ability to modify configuration and configuration of BeanSerializer instances. This makes it possible to do all kinds of modifications (changing order in which properties are serialized; adding, removing or renaming properties; replacing serializer altogether with a custom instance and so on): you can completely re-wire or replace serialization of regular POJO ("bean") types.

This is achieved by adding a BeanSerializerModifier: the simplest way to do this is by usingModule interface. Details of using BeanSerializerModifier are more advanced topic; I hope to cover it separately in future. The basic idea is that BeanSerializerModifier instance defines callbacks that Jackson BeanSerializerFactory calls during construction of a serializer.

Right, so now I know what I need to use but there's no details on actually how to use it.

Classes of interest

The 3 classes of concern are:

org.codehaus.jackson.map.ser.BeanSerializerModifier
org.codehaus.jackson.map.Module
org.codehaus.jackson.map.ObjectMapper
BeanSerializerModifier

When extended this class provides 4 entry points which can be used to customize the serialization process:

changeProperties() - Implementations can add, remove or replace any of passed properties.
orderProperties() - Implementations can change ordering any way they like.
updateBuilder() - Implementations may choose to modify state of builder, or even completely replace it.
modifySerializer() - Implementations can modify or replace given serializer and return serializer to use.
Module

This is a simple interface used for registering extensions with ObjectMapper. The setupModule() method will be called during initialization and this provides an instance of Module.SetupContext. The method addBeanSerializerModifier() on that context object can then be used to register a custom BeanSerializerModifier.

ObjectMapper

This is the main class within Jackson for serializing and deserializing objects, the Module is registered with this object.

Usage

Firstly you extend BeanSerializerModifier and override the serialization process in whatever custom way you wish. For example you might wish to reorder the fields and also exclude some. The API for this is well documented, read the javadocs.

Next you need to wire your modifier class into the ObjectMapper and you do this by creating a Module and then registering that with the ObjectMapper using registerModule(). In the setupModule() method in your module you register the new BeanSerilizerModifier.

Example Code

To show how this works in practice I've put together a small project that implements a FilteringBeanSerializerModifier, which changes the serialization behaviour of ObjectMapper by filtering out fields on specified classes.

You can find the source code on GitHub here:

https://github.com/ultraflynn/jackson-custom-serialization
