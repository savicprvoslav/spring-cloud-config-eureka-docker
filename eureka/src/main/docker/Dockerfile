FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD eureka-0.0.1-SNAPSHOT.jar app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""
ENV spring_application_name=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar --spring.application.name=$spring_application_name  " ]