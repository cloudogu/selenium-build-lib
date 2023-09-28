![Cloudogu logo](https://cloudogu.com/images/logo.png)

selenium-build-lib
=======

Jenkins shared library that provides a toolset for UI tests with selenium.

## [Working with Selenium]

For Selenium Grid to work it is vital that a Docker network is supplied (while Zalenium above may work with its own).
If you don't want the hassle of creating a Docker network there is good news: The
[Docker network step](#Docker Network creation) plays well into your cards. 

### Usage

In your `Jenkinsfile`

```groovy
@Library('github.com/cloudogu/selenium-build-lib@versionTag') _

// ...
stage('UI Test') {
    withDockerNetwork() { networkName ->  
        withSelenium(networkName) { seleniumIp ->
            // Run your build, passing ${zaleniumIp} to it
        }
    }
}
```

Besides the Docker network name there are also a number of parameters that you may or may not pass to the step. 

| Parameter | optional? | Default | Description |
|-----------|-----------|---------|-------------|
|seleniumHubImage  | optional | 'selenium/hub' | the Selenium Docker container images to be used from hub.docker.com.|
|seleniumVersion   | optional | "3.141.59-zinc" | the Selenium Docker container image tag |
|workerImageFF     | optional | "selenium/node-firefox" | the Selenium Firefox worker node Docker container image. This matches automatically with the given Selenium hub version. |
|workerImageChrome | optional | "selenium/node-chrome" | the Selenium Chrome worker node Docker container image. This matches automatically with the given Selenium hub version.|
|firefoxWorkerCount| mandatory if no Chrome worker is used | 0 | the number of Firefox containers that should be started for the test |
|chromeWorkerCount | mandatory if no Firefox worker is used | 0 | the number of Chrome containers that should be started for the test |
|hubPortMapping    | optional | 4444 | the port under which the Selenium Hub should be available |
|debugSelenium     | optional | false | set to `true` if you want your Jenkins run log to be filled with debug output |

Please note that `firefoxWorkerCount` AND `chromeWorkerCount` must not contain a value of zero. That would render the
test unusable because there would no one to execute the tests. In this case the body will not be executed and a
`ConfigurationException` will be thrown.  

```groovy
// example for starting Selenium with a certain parameter with a groovy map.
// The rest of the parameters fallback to their defaults. 
withSelenium([ seleniumVersion : '3.14.0-p15' ]) { seleniumIp ->
        // Run your build, passing ${zaleniumIp} to it
 }
```

### Attaching Test Reports to the Jenkins Build Run

The resulting videos aside, you may be interested in attaching the actual test report to the Jenkins build run. Due to
the fact that the resulting test report varies under the chosen technology stack (read: different format and location)
the test developer is the one responsible to attach the test report to the build run. This is done as usual with the 
[Jenkins pipeline step `archiveArtifacts`](https://jenkins.io/doc/pipeline/tour/tests-and-artifacts/).

## [Docker Network creation]

It is possible (although not necessary) to explicitly work with docker networks. This library supports the automatic creation and removal of a bridge network with a unique name.

### Usage

`withSelenium` accepts now an optional network name the Selenium containers can attach to the given network. Conveniently a docker network can be created with this pipeline step which provides the dynamically created network name.

```
    withDockerNetwork { networkName ->
        def yourConfig = [:]
        withSelenium(yourConfig, networkName) {
            docker.image("foo/bar:1.2.3").withRun("--network ${network}") {
            ...
        }
    }

```

## Working with Truststores

Often comes a Truststore into play while working with Jenkins and Java. Jenkins can accommodate necessary certificates in its truststore so Java applications like Maven (and others too!) can successfully interact with other parties, like download artifacts from artifact repositories or transport data over the network. Even so, it may be necessary to provide these Java applications with the right certificates when otherwise encrypted communication would fail without doing so.

### Simple Truststore pipeline

For such circumstances this library provides a small snippet. The global `truststore` variable ensures that any truststore files which are copied in the process are also removed at the end of both `copy` and `use` actions.

In order to successfully provide a truststore to any Java process this sequence must be in order:

1. copy the truststore with `truststore.copy()`   
1. use the copied truststore with `truststore.use { truststoreFile -> }`

Here is a more elaborate example:
 
```
Library ('zalenium-build-lib') _

node 'master' {
    truststore.copy()
}
node 'docker' {
    truststore.use { truststoreFile ->
        javaOrMvn "-Djavax.net.ssl.trustStore=${truststoreFile} -Djavax.net.ssl.trustStorePassword=changeit"
    }
}
```

## Alternative Ways of Configuration

It is possible to supply a different truststore than the Jenkins one. Also it is possible to provide a different name in order to avoid filename collision:

```
Library ('zalenium-build-lib') _

node('master') {
    truststore.copy('/path/to/alternative/truststore/file.jks')
}
node('anotherNode') {
    //truststore.use ... as usual
}
```

# Examples

* [cloudogu/spring-petclinic](https://github.com/cloudogu/spring-petclinic/blob/548db42f320f0f9065876c588c93754beffacc36/Jenkinsfile) (Java / Maven)
* [cloudogu/nexus](https://github.com/cloudogu/nexus/blob/434c8c3ebef740cd887462aad1292971c46d883e/Jenkinsfile)  (JavaScript / yarn)
* [cloudogu/jenkins](https://github.com/cloudogu/jenkins/blob/3bc8b6ab406477e0c3bb232e05f745b1fc91ba70/Jenkinsfile)  (JavaScript / yarn)
* [cloudogu/redmine](https://github.com/cloudogu/redmine/blob/740dd3a99a8111c31e4ada1ccb3023d84d6d205f/Jenkinsfile)  (JavaScript / yarn)
* [cloudogu/scm](https://github.com/cloudogu/scm/blob/4f1c998425e175a7a52f97dff5b78e82f244a9bf/Jenkinsfile)  (JavaScript / yarn)
* [cloudogu/sonar](https://github.com/cloudogu/sonar/blob/3488a6e7e38ee5e2e8fde807de570028e15835a1/Jenkinsfile)  (JavaScript / yarn)

---
### What is the Cloudogu EcoSystem?
The Cloudogu EcoSystem is an open platform, which lets you choose how and where your team creates great software. Each service or tool is delivered as a Dogu, a Docker container. Each Dogu can easily be integrated in your environment just by pulling it from our registry. We have a growing number of ready-to-use Dogus, e.g. SCM-Manager, Jenkins, Nexus, SonarQube, Redmine and many more. Every Dogu can be tailored to your specific needs. Take advantage of a central authentication service, a dynamic navigation, that lets you easily switch between the web UIs and a smart configuration magic, which automatically detects and responds to dependencies between Dogus. The Cloudogu EcoSystem is open source and it runs either on-premises or in the cloud. The Cloudogu EcoSystem is developed by Cloudogu GmbH under [MIT License](https://cloudogu.com/license.html).

### How to get in touch?
Want to talk to the Cloudogu team? Need help or support? There are several ways to get in touch with us:

* [Website](https://cloudogu.com)
* [myCloudogu-Forum](https://forum.cloudogu.com/topic/34?ctx=1)
* [Email hello@cloudogu.com](mailto:hello@cloudogu.com)

---
&copy; 2020 Cloudogu GmbH - MADE WITH :heart:&nbsp;FOR DEV ADDICTS. [Legal notice / Impressum](https://cloudogu.com/imprint.html)
