FROM tomcat:8-jre8

# Copy the WAR file into Tomcat's webapps folder
COPY webapp/src/main/webapp.war /usr/local/tomcat/webapps

# Copy default applications from webapps.dist (optional)
RUN cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps
