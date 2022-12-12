FROM python:alpine

RUN apk add --update --no-cache --virtual .build_deps build-base libffi-dev
RUN pip install --upgrade pip
RUN pip install parsedmarc
COPY parsedmarc.ini /