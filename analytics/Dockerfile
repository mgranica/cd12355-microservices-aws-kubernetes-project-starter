FROM python:3.11-slim


RUN apt update -y && apt install -y --no-install-recommends \
    build-essential libpq-dev

WORKDIR /app

COPY requirements.txt /app/
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \
    pip install --no-cache-dir -r requirements.txt

COPY . /app

EXPOSE 5153

CMD ["python", "app.py"]