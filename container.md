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

                       
