FROM python:3.10-alpine

RUN mkdir app

WORKDIR /app
COPY requirements.txt /app

RUN pip install --upgrade pip && \
  pip3 install -r requirements.txt && \
  apk update && \
  apk add bash && \
  apk add vim

COPY . /app

EXPOSE 8000

CMD ["python3", "app.py"]