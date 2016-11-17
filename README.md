docker-maven-jdk-oracle
============

This image is based on the [docker-maven image](https://github.com/carlossg/docker-maven)

# What is Maven?

[Apache Maven](http://maven.apache.org) is a software project management and comprehension tool.
Based on the concept of a project object model (POM),
Maven can manage a project's build,
reporting and documentation from a central piece of information.


# How to use this image


## Run a single Maven command

For many simple projects, you may find it inconvenient to write a complete `Dockerfile`.
In such cases, you can run a Maven project by using the Maven Docker image directly,
passing a Maven command to `docker run`:

    docker run -it --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven:3.3-jdk-7 mvn clean install


# Reusing the Maven local repository

The local Maven repository can be reused across containers by creating a volume and mounting it in `/root/.m2`.

    docker volume create --name maven-repo
    docker run -it -v maven-repo:/root/.m2 maven mvn archetype:generate # will download artifacts
    docker run -it -v maven-repo:/root/.m2 maven mvn archetype:generate # will reuse downloaded artifacts

# Packaging a local repository with the image

The `$MAVEN_CONFIG` dir (default to `/root/.m2`) is configured as a volume so anything copied there in a Dockerfile at build time is lost.
For that the dir `/usr/share/maven/ref/` is created, and anything in there will be copied on container startup to `$MAVEN_CONFIG`.

To create a pre-packaged repository, create a `pom.xml` with the dependencies you need and use this in your `Dockerfile`.
`/usr/share/maven/ref/settings-docker.xml` is a settings file that changes the local repository to `/usr/share/maven/ref/repository`,
but you can use your own settings file as long as it uses `/usr/share/maven/ref/repository` as local repo.

    COPY pom.xml /tmp/pom.xml
    RUN mvn -B -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve

To add your custom `settings.xml` file to the image use

    COPY settings.xml /usr/share/maven/ref/

For an example, check the `tests` dir

# Running as non-root

Maven needs the user home to download artifacts to, and if the user does not exist in the image an extra
`user.home` Java property needs to be set.

For example, to run as user `1000` mounting the host' Maven repo

    docker run -v ~/.m2:/var/maven/.m2 -ti --rm -u 1000 -e MAVEN_CONFIG=/var/maven/.m2 maven mvn -Duser.home=/var/maven archetype:generate

# Building

Build with the usual

    docker build -t maven .