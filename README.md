# Approval Process
This library gives you the ability to implement a generalized approval process in your project where multi-level approval process is necessary, for example, you have a Human Resource Management System (HRMS) or a Customer Relationship Manangement System (CRMS) where approval of certain activity is found very often. 

This library is implemented using java 1.8 and will support any java version from 1.8 onwards. You can also extend its functionality and make it serve your needs.
Just a disclaimer before diving into the how to guide, this library is not available on maven central so follow the steps to add this into your current project.

# How to guide

1. clone the project on your local machine with git clone command
2. navigate to the project directory
3. inside the libs directory you will find the approval.jar
4. copy the jar file
5. create a folder location inside your project's resource directory, for example, /src/resources/libs/
6. paste the approval.jar file inside the libs directory
7. it is assumed that your are using a build tool for your project i.e. maven or gradle
8. if so, then add the dependency path inside your pom.xml for maven, or build.gradle file for a gradle project
9. update the project
10. you are good to go!

# As a maven dependency

```maven
<dependency>
			<groupId>approval.process</groupId>
			<artifactId>approval-process</artifactId>
			<version>1.0.0</version>
			<scope>system</scope>
			<systemPath>${project.basedir}/src/resources/libs/approval-process-1.0.0.jar</systemPath>
</dependency>
```
