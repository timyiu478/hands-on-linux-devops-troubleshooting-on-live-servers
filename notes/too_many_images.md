## "Woluwe": Too many images

### Problem

Description: A pipeline created a lot of Docker images locally for a web app. All these images except for one contain a typo introduced by a developer: there's an incorrect image instruction to pipe "HelloWorld" to "index.htmlz" instead of using the correct "index.html"
Find which image doesn't have the typo (and uses the correct "index.html"), tag this correct image as "prod" (rather than fixing the current prod image) and then deploy it with docker run -d --name prod -p 3000:3000 prod so it responds correctly to HTTP requests on port :3000 instead of "404 Not Found".

Root (sudo) Access: True

Test: curl http://localhost:3000 should respond with HelloWorld;529

### Solution

