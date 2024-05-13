
FROM python:3.8-slim-buster
MAINTAINER Brahim Haouchine
COPY ./requirements.txt / ./student_age.py /

RUN apt-get update -y \
    && apt-get install -y --no-install-recommends \
        python3-dev \
        libsasl2-dev \
        libldap2-dev \
        build-essential \
        libatlas-base-dev \
        libssl-dev \
    && pip install --upgrade pip \
    && pip install -r requirements.txt \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean \
    && rm -rf /root/.cache/pip \
    && apt-get purge -y --auto-remove build-essential \
    && rm -rf /requirements.txt

VOLUME /data

EXPOSE 5000

CMD ["python", "./student_age.py"]

