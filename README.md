# AQCU Base Docker

[![Build Status](https://travis-ci.org/USGS-CIDA/docker-aqcu-base.svg?branch=master)](https://travis-ci.org/USGS-CIDA/docker-aqcu-base)

This Docker Image provides basic functionality common to all AQCU Service Docker Images.

## Usage
### Environment Variables
This docker image provides several environment variables to child images:
 - **requireSsl** [deafult: true]: Whether or not the service should disallow connections over plain HTTP.
 - **serverPort** [default: 443]: The port that the application should run on within the container.
 - **serverContextPath** [default: /]: The root context that the application should run on within the container.
 - **springFrameworkLogLevel** [deafult: info]: The logging level of the application running within the container.
 - **keystoreLocation** [default: /localkeystore.p12]: The fully-qualified file path that the keystore should be created at.
 - **keystorePassword** [default: changeme]: The password to use for the keystore.
 - **keystoreSSLKey** [default: tomcat]: The key alias to use for the SSl certicate that should be served by the application.
 - **ribbonMaxAutoRetries** [default: 3]: The number of times that connections to other services via ribboning should retry before failing.
 - **ribbonConnectTimeout** [default: 1000]: The amount of time to wait before timing out a ribbon connection attempt (in MS).
 - **ribbonReadTimeout** [default: 10000]: The amount of time to wait before timing out while waiting on a response from an established ribbon connection (in MS).
 - **hystrixThreadTimeout** [default: 10000000]: The amount of time hystrix should allow threads to run before timing them out (in MS).
 - **SPRING_CLOUD_CONFIG_ENABLED** [default: false]: Wherher or not the application should attempt to connect to a Spring Cloud Config Server.
 - **TOMCAT_CERT_PATH** [default: /tomcat-wildcard-ssl.crt]: The path where the tomcat cert file is expected to be mounted.
 - **TOMCAT_KEY_PATH** [default: /tomcat-wildcard-ssl.key]: The path where the tomcat key file is expected to be mounted.
 - **HEALTHY_STATUS** [default: '{"status":"UP"}']: The string to expect to be present in the application health check response in order to indicate that the service is healthy.
 - **HEALTH_CHECK_URL** [default: "https://127.0.0.1:${serverPort}${serverContextPath}/health"]: The URL that Docker should hit to reach the application health check.

### Using Mounted Files
As mentioned above, the `TOMCAT_CERT_PATH` and `TOMCAT_KEY_PATH` variables should be the paths to two mounted files. These files can be mounted into your container in several ways, two of which are listed below.

#### Docker Secrets
The `docker-compose-default.yml` file included in this repo illustrates how these files can be mounted into your container using Docker Secrets.

#### Docker Mount Command
When launching your docker container you can specify files and directories to mount into your container. If you mount a directory to your container containing the cert and key files and it is mounted somewhere other than `/` then you must override the values for `TOMCAT_CERT_PATH` and `TOMCAT_KEY_PATH` to point to the location where the files do get mounted.

### Extending the Base Image
This docker image is not intended to be run on its own, rather other images should inherit from this image in order to utilize its functionality. 

#### The Entrypoint Script
This docker image includes an entrypoint script that configures the keystore properly in order for your application to serve SSL certificates. Additionally, this entrypoint allows for your application to be launched via an additional script (detailed in the next section) if you need to run it with specific arguments or additional configuration. This entrypoint script is added to the docker image as `/entrypoint.sh`. 

Dockerfiles which extend this base image should _**not**_ set their own entrypoint in the Dockerfile _**unless**_ your entrypoint calls the base entrypoint script, recreates the base entrypoint behavior, or your application has its own specific configuration for the keystore. 

If you do choose to override the entrypoint but want to call the base entrypoint script from within your entrypoint you must mount your entrypoint somewhere other than `/entrypoint.sh` to be sure that you do not overwrite the base entrypoint script.

#### Executing your Application
The entrypoint script added by this docker image expects your application to be launched in one of two pre-defined ways. If you cannot support one of these two methods for some reason then you should override the entrypoint script with your own.

1. **/launch-app.sh**: If you add a shell script to the path `/launch-app.sh` the entrypoint script will execute that script after setting up the keystore. Within this script your can complete any additional configuration that you may need and launch your JAR file or other artifact.

2. **/app.jar**: If you do not provide a `/launch-app.sh` script then the entrypoint will attempt to directly launch an application JAR file that has been placed at `/app.jar`. This jar file is executed using the following command: 
    ```
    java -Djava.security.egd=file:/dev/./urandom -jar -DkeystorePassword=$keystorePassword app.jar $@
    ```

If neither of the above files are provided in your docker image and you do not override the entrypoint script then the following message will appear and the docker container will exit: `No /launch-app.sh or /app.jar found. Exiting.`


#### Overriding Environment Variables
Any of the environment variables listed above can be overridden in the child docker image by providing a new value for the variable. This value can be provided as an **ENV** value in the Dockerfile itself, as a value in a docker-compose ***.env** file, or via any other method that you might go about assigning a value to an environment variable.
