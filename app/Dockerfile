FROM docker.io/library/python:3.11.3-alpine
ENV PYTHONUNBUFFERED 1

ENV DJANGO_DEBUG=False

RUN apk update && apk upgrade
RUN python -m pip install gunicorn

COPY . /app
WORKDIR /app
RUN python -m pip install -r requirements.txt

EXPOSE 8000

CMD ["gunicorn", "--bind", ":8000", "--workers", "3", "demo.wsgi:application"]