FROM python

RUN apt-get update -y && \
    apt-get install -y python-pip- python-dev

    
RUN pip install pyyaml

COPY . .

EXPOSE 3000

CMD [ "python", "app.py" ]