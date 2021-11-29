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
The client subproject contains the code for, you guessed it, the Mirth Administrator. That includes the code for the bare UI itself, aswell as code for different connectors and other plugins. The authors of Mirth are clever in the sense that they make all their official additions in the form of plugins, so plugin interfaces are kept up to date.

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

So... `git clone git@github.com:nextgenhealthcare/connect.git`. Now open the resulting folder up in your favorite IDE. I'm going to use [IntelliJ](https://www.jetbrains.com/idea/) cause I like that one.

Another good idea is to clone my [Mirth sample plugin](https://github.com/kpalang/mirth-sample-plugin) to get a general idea of the plugin file tree. Also we will be using Maven to build the project, so that's something to look up if you haven't already.

And finally I've made a [Docker image](https://hub.docker.com/r/kpalang/connect/) that includes a JVM option that enables remote debugging. This will come handy later, but now I'm going to set up a Docker container. The container will be useful for quick runs to check if the plugin works or not. It's also all sandboxed so you pretty much need one command to run it and another one to stop it. I've included a sample [docker-compose.yaml](/docker-compose.yaml).

---

## 1 - Getting started
The part you've been waiting for. I'm going to clone [my sample plugin](https://github.com/kpalang/mirth-sample-plugin), because it already has some of the work done for us: `git clone git@github.com:kpalang/mirth-sample-plugin.git`

The sample plugin structure consists of four main parts:
1. client
1. distribution
1. server
1. shared

Client module (as in Maven lingo) will get all the code that as to do with the UI. Distribution module is not actually related to the functionality of our plugin, but is actually needed to make our life packaging that plugin a tad easier. Server module get all the backend bits of our code. And finally, shared module gets all those bits that are needed on both sides of our plugin, UI and backend.

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

![Details in Mirth Administrator](images/admin_extension_details.png)

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

After all this initial tweaking I've come to a state as in [this commit](https://github.com/kpalang/mirth-sample-plugin/tree/d3b36ea63a893332f6e0eda5283d88f00e8dc25c) (_I modified some other stuff aswell but that's unimportant right now..._).</br> **NOTE:** Your current state may differ a bit depending on when you clone the repository.

---

## 2 - Writing serverside code

The easiest way to get that _ofmygoditworks_ kick is to go into `server` module and edit `MyServicePlugin.java`.

```java
package com.kaurpalang.mirthpluginsample.server;

import com.kaurpalang.mirth.annotationsplugin.annotation.ServerClass;
import com.kaurpalang.mirthpluginsample.shared.Constants;
import com.mirth.connect.model.ExtensionPermission;
import com.mirth.connect.plugins.ServicePlugin;

import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

@ServerClass
public class MyServicePlugin implements ServicePlugin {

    @Override
    public void init(Properties properties) {
        System.out.println("Hello world from init!");
    }

    @Override
    public void update(Properties properties) {
        // We don't need to do anything here.
    }

    @Override
    public Properties getDefaultProperties() {
        return new Properties();
    }

    @Override
    public ExtensionPermission[] getExtensionPermissions() {
        return new ExtensionPermission[0];
    }

    @Override
    public Map<String, Object> getObjectsForSwaggerExamples() {
        return new HashMap<>();
    }

    @Override
    public String getPluginPointName() {
        return Constants.PLUGIN_POINTNAME;
    }

    @Override
    public void start() {
        System.out.println("Hello world from start!");
    }

    @Override
    public void stop() {
        System.out.println("Good bye world!");
    }
}
```
You'll notice in the function `getPluginPointName()`, I've referenced a `Constants` class from the `shared` module. This is so we can reference the same... _constants_, in both server- and client-side code.

After editing the strings in this class, build the plugin with `mvn clean package`, install the plugin, the zip file will be in `distribution/target/` on your server and restart said server.</br>
If you used the container from above, your logs should look something like this:
```
Found 1 custom extensions.
Listening for transport dt_socket at address: 5005
Hello world from init!
Hello world from start!
INFO  2021-11-26 12:54:09,496 [Main Server Thread] com.mirth.connect.server.Mirth: Mirth Connect 3.12.0 (Built on September 2, 2021) server successfully started.
INFO  2021-11-26 12:54:09,498 [Main Server Thread] com.mirth.connect.server.Mirth: This product was developed by NextGen Healthcare (https://www.nextgen.com) and its contributors (c)2005-2021.
INFO  2021-11-26 12:54:09,498 [Main Server Thread] com.mirth.connect.server.Mirth: Running Eclipse OpenJ9 VM 11.0.12 on Linux (5.4.0-90-generic, amd64), postgres, with charset UTF-8.
INFO  2021-11-26 12:54:09,499 [Main Server Thread] com.mirth.connect.server.Mirth: Web server running at http://10.10.1.3:8080/ and https://10.10.1.3:8443/
```
...

![Hackerman gif](/images/hackerman.gif)

---

## 3 - Preparing for communcations with UI (aka REST API)

_Note. [I refactored `Constants` to `MyConstants`](https://github.com/kpalang/mirth-sample-plugin/commit/568ef245a204dae3ffa1ae23efa9c862073d555e) to get more clarity between this and the one in mirth-plugin-maven-plugin._

Right, now that you've got your hacking gloves on, lets start making our plugin controllable by adding API endpoints.

First we need an object to transfer. We _could_ use just a String, but that way we won't see the automatic serialisation of POJOs. I chose to create a `MyInfoObject.java` class. This class has to go into the `shared` module, because that object has to be available to both server and client parts of our plugin. I also chose to place it in `com.kaurpalang.mirthpluginsample.shared.model` package, just to make the codebase more readable.

So, the POJO:
```Java
package com.kaurpalang.mirthpluginsample.shared.model;

public class MyInfoObject {
    private String data;

    public MyInfoObject(String data) {
        this.data = data;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }
}
```

Ooor... _hint, hint..._ [lombok](https://projectlombok.org/)
```java
package com.kaurpalang.mirthpluginsample.shared.model;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@AllArgsConstructor
public class MyInfoObject {
    @Getter @Setter private String data;
}
```

Mirth's API mechanic comprises mostly of two parts. A serverside servlet class and a shared servlet interface class. The servlet class is the one actually doing the work, therefore it has to run on the server. Servelet interface on the other hand has to be shared because the client will call servlet interface classes directly. This wasy we can also utilize XStream for pojo (de-)serialization.

```java
package com.kaurpalang.mirthpluginsample.shared.interfaces;

import com.kaurpalang.mirth.annotationsplugin.annotation.ApiProvider;
import com.kaurpalang.mirth.annotationsplugin.type.ApiProviderType;
import com.kaurpalang.mirthpluginsample.shared.MyPermissions;
import com.kaurpalang.mirthpluginsample.shared.model.MyInfoObject;
import com.mirth.connect.client.core.ClientException;
import com.mirth.connect.client.core.Operation;
import com.mirth.connect.client.core.api.BaseServletInterface;
import com.mirth.connect.client.core.api.MirthOperation;
import com.mirth.connect.client.core.api.Param;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;

import javax.ws.rs.*;
import javax.ws.rs.core.MediaType;

@Path("/myplugin")
@Tag(name = "MyPlugin operations")
@Consumes({ MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON })
@Produces({ MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON })
@ApiProvider(type = ApiProviderType.SERVLET_INTERFACE)
public interface MyServletInterface extends BaseServletInterface {

    @GET
    @Path("/getsomething")
    @Consumes({ MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON })
    @Produces({ MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON })
    @ApiResponse(responseCode = "200", description = "Found the information",
            content = {
                    @Content(mediaType = MediaType.APPLICATION_JSON, schema = @Schema(implementation = MyInfoObject.class)),
                    @Content(mediaType = MediaType.APPLICATION_XML, schema = @Schema(implementation = MyInfoObject.class))
            })
    @MirthOperation(
            name = "getSomething",
            display = "Get important information",
            permission = MyPermissions.GETSTH,
            type = Operation.ExecuteType.ASYNC
    )
    MyInfoObject getSomething(
            @Param("identifier") @Parameter(description = "The identifier of our important information to retrieve.", required = true) @QueryParam("identifier") String identifier)
            throws ClientException;
}
```

Lets go over this one-by-one:

#### Swagger stuff

`@Path("/myplugin")`</br>
This annotation sets parent path of our endpoints:</br>
`https://localhost:8443/api/myplugin/`

--</br>
`@Consumes({ MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON })`</br>
This annotation tells Swagger our endpoint can consume either XML or JSON inputs</br>

--</br>
`@Produces({ MediaType.APPLICATION_XML, MediaType.APPLICATION_JSON })`</br>
This one tells Swagger out endpoint produces either XML or JSON</br>

#### Not Swagger stuff

`@ApiProvider(type = ApiProviderType.SERVLET_INTERFACE)`</br>
This line tells mirth-plugin-maven-plugin to include this class in our plugin's `plugin.xml` as the following line, which in turn tells Mirth to handle our class as a servlet interface.</br>
```xml
<apiProvider name="com.kaurpalang.mirthpluginsample.shared.interfaces.MyServletInterface" type="SERVLET_INTERFACE"/>
```
--</br>
`public interface MyServletInterface extends BaseServletInterface`</br>
BaseServletInterface is Mirth's BaseServletInterface... interface... It creates a foundation to base our servlet interfaces on...

--</br>
More Swagger stuff we touched just above

--</br>
```java
@ApiResponse(
  responseCode = "200",
  description = "Found the information",
  content = {
          @Content(mediaType = MediaType.APPLICATION_JSON, schema = @Schema(implementation = MyInfoObject.class)),
          @Content(mediaType = MediaType.APPLICATION_XML, schema = @Schema(implementation = MyInfoObject.class))
})
```
This part tells Swagger to use our POJO class as on output example. Also, automagic serialization ;) I'll show you the picture in a second.

--</br>
```java
@MirthOperation(
    name = "getSomething",
    display = "Get important information",
    permission = MyPermissions.GETSTH
)
```
This annotation dictates first, who can execute it. TBH I don't really know right now what the permission parameter does, but I'd guess it allows for permission control if you've got the premium User Roles plugin installed.
Also you can see I've put the permission string into [it's own class](https://github.com/kpalang/mirth-sample-plugin/blob/main/shared/src/main/java/com/kaurpalang/mirthpluginsample/shared/MyPermissions.java).</br>
And secondly, this creates event messages in Events view in Mirth Administrator.

![Events view with our custom event](/images/admin_event.png)

--</br>
```java
MyInfoObject getSomething(
            @Param("identifier") @Parameter(description = "The identifier of our important information to retrieve.", required = true) @QueryParam("identifier") String identifier)
            throws ClientException;
```
Finally the function itself. Geez. Well it specifies what does in and what comes out of our function...

#### Grand middle finale
If you've done everything more or less like I've done above, you should see something like this:
![Swagger view](/images/swagger_endpoint_look.png)

I know right...

<img src="/images/satisfying.jpg" alt="Satisfying" width="150"/>
</br>
</br>
</br>

Okay now, stop rubbing yourself. We've still got to implement the interface.
```java
package com.kaurpalang.mirthpluginsample.server.servlet;

import com.kaurpalang.mirth.annotationsplugin.annotation.ApiProvider;
import com.kaurpalang.mirth.annotationsplugin.type.ApiProviderType;
import com.kaurpalang.mirthpluginsample.shared.MyConstants;
import com.kaurpalang.mirthpluginsample.shared.interfaces.MyServletInterface;
import com.kaurpalang.mirthpluginsample.shared.model.MyInfoObject;
import com.mirth.connect.server.api.MirthServlet;

import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.SecurityContext;

@ApiProvider(type = ApiProviderType.SERVER_CLASS)
public class MyPluginServlet extends MirthServlet implements MyServletInterface {

    public MyPluginServlet(@Context HttpServletRequest request, @Context SecurityContext sc) {
        super(request, sc, MyConstants.PLUGIN_POINTNAME);
    }

    @Override
    public MyInfoObject getSomething(String identifier) {
        String data = String.format("<%s> Some important informations", identifier);
        return new MyInfoObject(data);
    }
}
```
To be honest, only the `@ApiProvider` annotation deserves attention here. Notice the type is now `ApiProviderType.SERVER_CLASS`, as opposed to `ApiProviderType.SERVLET_INTERFACE` in `MyServletInterface`. It tells mirth-plugin-maven-plugin to put this line in our plugin definition file which tells Mirth this class is a Servlet class and that it should process this:
```XML
<apiProvider name="com.kaurpalang.mirthpluginsample.server.servlet.MyPluginServlet" type="SERVER_CLASS"/>
```

Right. Now you should be in a state [something like this](https://github.com/kpalang/mirth-sample-plugin/tree/d0029888ca978bbb9b8dabdd04446a8b408510c9).

---

## ? - Signing and publishing
Mirth Connect plugins need to be signed with a code-signing certificate. This certificate can be either self-signed or bought from a proper certificate authority. There have been some debate over which certificate authorities are trusted by Mirth. I've had success with [DigiCert](https://www.digicert.com/).

We are going to use a self-signed certificate to sign our plugin because official certificates cost a couple hundred dollars. But fret not, for a self-signed works for our current purpose of getting the plugin running on Mirth.
