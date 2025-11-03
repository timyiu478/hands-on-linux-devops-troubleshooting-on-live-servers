## "Cabedelo": Harbor full of issues

### Problem

Description: You need to build and push a docker image **without changing the Dockerfile** to your company's Harbor registry, which is running at harbor.sadservers.local, with its home directory at /opt/harbor. You have full admin access with admin:Harbor12345 credential. The source code and the Dockerfile are in the ~/app directory. The image name must be harbor.sadservers.local/images/app:1.0.0. It is also expected that the application will be up and running at localhost:5000 in a container named app.

IMPORTANT. Do not:

1. Generate new internal certificates
2. Change the Dockerfile
3. Change the /opt/harbor.yml file

Root (sudo) Access: True

Test: You are able to pull the application image from Harbor:
docker rmi harbor.sadservers.local/images/app:1.0.0
docker pull harbor.sadservers.local/images/app:1.0.0


You can access the application; curl localhost:5000 returns Hello world!

### Solution

todo
