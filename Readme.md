# Unofficial Mirth plugin guide

This is an unofficial collection of tips and tricks on the topic of Mirth Connect plugin creation. Enjoy!

---

## Foreword
If you, the reader, have any suggestions, questions, or anything at all you'd like to add, please either create a pull-request or an Issue and I'll look into it.

Also I'm going to be use _**Mirth**_ to refer to [this product](https://github.com/nextgenhealthcare/connect/), regardless of how it is officially named at the time.

---

## General overview

Mirth repository consists of eight sub-projects:
- client
- command
- donkey
- generator
- manager
- server
- simplesender
- webadmin

### Client
The client subproject contains the code for, you guessed it, the Mirth Administrator. That includes the code for the bare UI itself, aswell as code for different connectors and other plugins. the authors of Mirth are clever in the sense that they make all their official additions in the form of plugins, so plugin interfaces are kept up to date.

### Command
This subproject contains code for the commandline interface, or CLI. As you can see there's not much there.

### Donkey
Donkey is the foundation on which Mirth is built. It's a collection of core models, some utility classes, part of the server's inner workings, database classes, and such. Plugin writers don't usually have to add stuff here.

### Generator
_To be fleshed out_

### Manager
_To be fleshed out_

### Server
The server subproject includes the _servery_ part of the Mirth product. This includes data and message manipulation, message parsing, and all that good stuff.

### SimpleSender
_To be fleshed out_

### WebAdmin
_To be fleshed out_

---

## Requirements
- Java JDK - I suggest version 1.8 because the UI part of Mirth uses that. I'm going to use a [Temurin build](https://adoptium.net/index.html?variant=openjdk8&jvmVariant=hotspot). You can get installation instructions for your OS version at the bottom of that page. Since I'm using Linux I'm going to install that from my package manager.
- Maven - You can download a version of Maven [here](https://maven.apache.org/index.html). I'm going to use version 3.8.3.

We will also be using another neat tool of mine, a [mirth-plugin-maven-plugin](https://github.com/kpalang/mirth-plugin-maven-plugin), which will save us some manual work later on. You don't have to do anything with it right now.

---

## 0 - Preparation
A fair first place would be to clone yourself a copy of [Mirth Connect](https://github.com/nextgenhealthcare/connect/). That way you can take a look at the structure and inheritance of Mirth classes.

So... `git clone git@github.com:nextgenhealthcare/connect.git`. Now open the resulting folder up in your favourite IDE. I'm going to use [IntelliJ](https://www.jetbrains.com/idea/) cause I like that one.

-Here will be a quick look around the structure-

Another good idea is to clone my [Mirth sample plugin](https://github.com/kpalang/mirth-sample-plugin) to get a general idea of the plugin file tree. Also we will be using Maven to build the project, so that's something to look up if you haven't already.

---

## 1 - Getting started
The part you've been waiting for. I'm going to clone [my sample plugin](https://github.com/kpalang/mirth-sample-plugin), because it already has some of the work done for us: `git clone git@github.com:kpalang/mirth-sample-plugin.git`

The sample plugin structure consists of four main parts:
1. client
1. distribution
1. server
1. shared

Client module (as in Maven lingo) will get all the code that as to do with the UI. Distribution module is not actually related to the functionality of our plugin, but is actually needed to make our life packaging that plugin a tad easier. Server module get all the backend bits of our code. And finally, shared module gets all those bits that are needed on boths sides of our plugin, UI and backend.

Additionally, you will find a `certificate` folder, which contains a readymade self-signed certificate that will sign our plugin in the name of BigCompany located in Big City, and a `libs` folder. Now `libs` folder will contain the libraries we want to include in our project separated by times. Anything we want to use at compiletime will go into `libs/compiletime`, and anything we need at runtime, when our plugin is running already, will go into `libs/runtime`.

You will also see a `pom.xml` file in your project root. This is our Maven project's definition. The general structure of `pom.xml` is not in the scope of this guide, but let's go over the relevant bits.


```xml
<properties>
    <!--
    We're using a custom annotation processor from
    the mirth-plugin-maven-plugin and this line
    specifies a place where that processor stores it's inner data.
    -->
    <processor.aggregator.path>distribution/aggregator/aggregated.json</processor.aggregator.path>


    <!--
    Here are plugin details.

    Some of these will be shown in
    the extensions view in Mirth Administrator.
    -->

    <!-- this is the name of the folder
    that gets put into mirthroot/extensions -->
    <plugin.path>plugintest</plugin.path>

    <!-- The name of your plugin -->
    <plugin.name>Sample testing plugin</plugin.name>

    <!-- Would you like to display your homepage? -->
    <plugin.url>www.com</plugin.url>

    <!-- The name of the author. You, your company, anything you like really -->
    <plugin.author>Kaur Palang</plugin.author>

    <!-- What version would you like to display in Mirth Administrator -->
    <plugin.version>15</plugin.version>

    <!--
    The version of Mirth your plugin is compatible with.
    This can also be a comma-separated list:
    3.10.0, 3.10.1, 3.11.0
    -->
    <plugin.mirthVersion>3.12.0</plugin.mirthVersion>

    <!-- This property sets the name of your eventual zipfile -->
    <plugin.archive.name>sampleplugin</plugin.archive.name>
</properties>
```

![Details in Mirth Administrator](images/admin_plugin_details.png)

```xml
<!--
This bit specifies a Nexus repository I'm
hosting for Mirth jarfiles so we don't have
to manually extract them from Mirth itself.
-->
<repositories>
    <repository>
        <id>nexus</id>
        <url>https://nexus.kaurpalang.com/repository/maven-public/</url>
    </repository>
</repositories>

<dependencies>
    <!-- Helper plugin to handle Mirth plugin specific tasks. -->
    <dependency>
        <groupId>com.kaurpalang</groupId>
        <artifactId>mirth-plugin-maven-plugin</artifactId>
        <version>${mirth-plugin-maven-plugin.version}</version>
    </dependency>
</dependencies>
```

The last thing we'll cover is the following. This bit makes sure our plugin get signed and sets the required parameters.
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jarsigner-plugin</artifactId>
    <version>${maven-jarsigner-plugin.version}</version>
    <executions>
        <execution>
            <id>sign</id>
            <goals>
                <goal>sign</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- Path to our keystore -->
        <keystore>${project.parent.basedir}/certificate/keystore.jks</keystore>
        <!-- The name of our certificate -->
        <alias>selfsigned</alias>
        <!-- Password to access the keystore -->
        <storepass>storepass</storepass>
        <!-- Password for the certificate -->
        <keypass>keypass</keypass>
    </configuration>
</plugin>
```

You don't have to touch anything else for now.

Now let's run `mvn clean package` to make sure our plugin builds properly. You should see `distribution/target/sampleplugin.zip` pop up. Take a peek inside to get an idea of what the archive will look like. I'm not going to cover it here.



---

## ? - Signing and publishing
Mirth Connect plugins need to be signed with a code-signing certificate. This certificate can be either self-signed or bought from a proper certificate authority. There have been some debate over which certificate authorities are trusted by Mirth. I've had success with [DigiCert](https://www.digicert.com/).

We are going to use a self-signed certificate to sign our plugin because official certificates cost a couple hundred dollars. But fret not, for a self-signed works for our current purpose of getting the plugin running on Mirth.
