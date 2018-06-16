apiVersion: v1
kind: Pod
metadata:
  name: mytomcat
  labels:
    env: development
spec:
  containers:
  - name: rawtomcat
    image: tomcat:8.0.52-jre8
