# Container  samples for hands-on

vim  my-app.py


```
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Python Web App runned by yomex 🚀"

app.run(host="0.0.0.0", port=5000)

```

vim  Dockerfile

```
FROM python:3.10-slim

WORKDIR /app

COPY my-app.py .

RUN pip install flask

EXPOSE 5000

CMD ["python", "my-app.py"]

```

build the dockerfile to image 

```
 docker build -t my-flask-app .
```

 common docker command 

```
docker ps
```

getting container & image informations
 ```
 docker ps -a
```

getting image informations
```
docker images
```

getting specific image informations
```
 docker images | grep my-flask-app  
```

run the image
```
 docker run -d -p 5000:5000 --name my-flask  my-flask-app 
```

check again to see if the image exist

```
docker ps
```
stop 
```
docker stop c16f893224e4
```

test the container 
```
curl http://127.0.0.1:5000 
```

remove 
```
docker rm c16f893224e4 
```

check again
```
docker ps
```

## interact with shell

```
docker exec -it 2d6633592634 sh
```

1. Shell built-ins
   
 ```
ls
 ```
 ```
pwd
 ```
 ```
echo $PATH
 ```
 ```
cd /app
 ```

```
hostname -i
```


3. Python (your container has Python)

```
python
python3 my-app.py
```

5. View files


 ```
cat my-app.py
ls -al
```

7. Run scripts or commands that exist in the image


 ```
pip list
python -m http.server 8000
```

                       
