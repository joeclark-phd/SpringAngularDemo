# SpringAngularDemo - modified for Tomcat 9

Marco Molteni's demo [Deploy Angular with Spring Boot in the same executable JAR](https://marcomolteni.ch/angular-with-java) is great, but it stops with a standalone JAR.  I want to use the same technique with my application which must be deployed in a Tomcat 9 container, so I began by modifying his demo code.

## Changes

[With help from Stackoverflow user "Pino"](https://stackoverflow.com/a/77366372), I made these changes:

- Added `<packaging>war</packaging>` to `/delivery/pom.xml`.

- Modify the main class so it `extends SpringBootServletInitializer`.

- "Hide" the embedded Tomcat dependency by marking it "provided".  I did that by adding the following to `/delivery/pom.xml`:

      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-tomcat</artifactId>
          <scope>provided</scope>
      </dependency>


The result is a WAR file that should work within a Tomcat 10+ container at the server root.  It can *also* still work as a standalone JAR (just run `java -jar MyBeautifulApp-0.0.1-SNAPSHOT.war`).  

## More Changes

I have a few other constraints, though, so I also made the following changes which may be "optional" for someone else:

- I want a consistent name for my WAR file, so I set `<finalName>${project.artifactId}</finalName>` in `/delivery/pom.xml`.  This will result in a file name "delivery.war".

- When I drop "delivery.war" into Tomcat's `/webapps`, the application will run at `http://localhost:8080/delivery` by default.  I need to tell the Angular front-end that its base URL starts with "/delivery" so it can find its various web assets.

  - Modify `package.json`'s "build" script to: 

        "build": "ng build --base-href delivery",

  - Change the second "execution" step in `/frontend/pom.xml` to call `npm run build` instead of `ng build`.  That section now contains:

        <configuration>
            <executable>npm</executable>
            <arguments>
                <argument>run</argument>
                <argument>build</argument>
            </arguments>
        </configuration>

- Finally, I am required to deploy to Tomcat 9.  Spring 3 requires Tomcat 10+.  Therefore, I had to downgrade my Spring Boot version in the parent `/pom.xml`.  Fortunately, this didn't break anything.  If you have Tomcat 10+, you don't have to do this:

      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.6.2</version>
      </parent>

