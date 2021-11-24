# Unofficial Mirth plugin guide

This is an unofficial collection of tips and tricks on the topic of Mirth Connect plugin creation. Enjoy!

---

## Foreword
If you, the reader, have any suggestions, questions, or anything at all you'd like to add, please either create a pull-request or an Issue and I'll look into it.

Also I'm going to be use _**Mirth**_ to refer to [this product](https://github.com/nextgenhealthcare/connect/), regardless of how it is oficially named at the time.

Now let's get on to it.

---

## Requirements
- Java JDK - I suggest version 1.8 because the UI part of Mirth uses that. I'm going to use a [Temurin build](https://adoptium.net/index.html?variant=openjdk8&jvmVariant=hotspot). You can get installation instructions for your OS version at the bottom of that page.
- Maven - You can download a version of Maven [here](https://maven.apache.org/index.html). I'm going to use version 3.8.4.

We will also be using another neat tool of mine, a [mirth-plugin-maven-plugin](https://github.com/kpalang/mirth-plugin-maven-plugin), which will save us some manual work later on. You don't have to do anything with it right now.

---

## 0 - Preparation
A fair first place would be to clone yourself a copy of [Mirth Connect](https://github.com/nextgenhealthcare/connect/). That way you can take a look at the structure and inheritance of Mirth classes.

So... `git clone git@github.com:nextgenhealthcare/connect.git`. Now open the resulting folder up in you favourite IDE. I'm going to use [IntelliJ](https://www.jetbrains.com/idea/) cause I like that one.

-Here will be a quick look around the structure-

Another good idea is to clone my [Mirth sample plugin](https://github.com/kpalang/mirth-sample-plugin) to get a general idea of the plugin file tree. Also we will be using Maven to build the project, so that's something to look up if you haven't already.

---

## 1 - Getting started


---

## #? - Signing and publishing
Mirth Connect plugins need to be signed with a code-signing certificate. This certificate can be either self-signed or bought from a proper certificate authority. There have been some debate over which certificate authorities are trusted by Mirth. I've had success with [DigiCert](https://www.digicert.com/).

We are going to use a self-signed certificate to sign our plugin because official certificates cost a couple hundred dollars. But fret not, for a self-signed works for our current purpose of getting the plugin running on Mirth.
