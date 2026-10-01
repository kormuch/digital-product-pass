FROM python:3.12-slim

WORKDIR /app
COPY app ./app
COPY static ./static
COPY config ./config
COPY docs ./docs
COPY tests ./tests

RUN pip install --no-cache-dir fpdf2

ENV HOST=0.0.0.0
ENV PORT=8080
ENV THEBEN_DATA_DIR=/app/data

EXPOSE 8080
CMD ["python", "-m", "app.app"]
