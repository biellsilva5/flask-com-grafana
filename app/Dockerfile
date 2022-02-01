FROM python:3-alpine

ADD requirements.txt /requirements.txt

RUN pip install -r /requirements.txt

ADD app.py /app.py

ENV FLASK_APP /app.py

CMD flask run --host=0.0.0.0