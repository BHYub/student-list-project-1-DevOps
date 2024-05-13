
FROM python:3.8-buster
MAINTAINER Brahim Haouchine
COPY ./requirements.txt /

RUN apt-get update -y && \
    apt-get install -y \
    python-dev \
    python3-dev \
    libsasl2-dev \
    libldap2-dev \
    libssl-dev \ 
    && rm -rf /var/lib/apt/lists/* \
    && pip install -r requirements.txt \
    && rm -rf pip requierements.txt \
    && apt-get clean 

VOLUME /data
COPY ./student_age.py /

EXPOSE 5000

CMD ["python", "./student_age.py"]

