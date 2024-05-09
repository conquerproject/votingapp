# Base Image
FROM debian:stable-slim AS base

RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 python3-pip python3-venv && \
    rm -rf /var/lib/apt/lists/*
    
WORKDIR /app

COPY requirements.txt .

RUN python3 -m venv venv
ENV PATH="/app/venv/bin:$PATH"

RUN pip3 install --no-cache-dir -r requirements.txt

FROM base AS dev

COPY . .

EXPOSE 80

RUN useradd -m app
USER app

ENTRYPOINT [ "python3" ]
CMD [ "app.py" ]
