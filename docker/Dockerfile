FROM python:3.12-slim

WORKDIR /app

COPY app.py /app/

# Instalujemy curl i Flask
RUN apt-get update && \
    apt-get install -y curl && \
    pip install --no-cache-dir flask && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 5001

CMD ["python", "app.py"]

