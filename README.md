Code to build dataProbeAdminApp.war using Gradle
===========================

Follwing changes made to support Gradle folder Structure
 1. moved src/com to src/main/java
 2. moved WebContent to src/main/webapp
 
To run build in your local, Please follow below steps:
  1. Checkout the code, open command prompt and go to checkout code location
  2. run below command
        gradlew.bat -DartifactoryUser="<your ibm mail id>" -DartifactoryPassword="<your ibm password or artifactory API key>" clean build
