FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC-3

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    git \
    python3 \
    python3-pip \
    python3-venv \
    python3-dev \
    unzip \
    wget \
    && rm -rf /var/lib/apt/lists/*

ENV VENV_PATH=/opt/venv
RUN python3 -m venv $VENV_PATH
ENV PATH="$VENV_PATH/bin:$PATH"

COPY requirements.txt requirements.txt
RUN pip install --upgrade pip && pip install -r requirements.txt

EXPOSE 5000

WORKDIR /root/workspace

CMD ["/bin/bash", "-c", "mlflow server --host 0.0.0.0 --port 5000 --backend-store-uri sqlite:////root/mlflow/mlruns.db --serve-artifacts --default-artifact-root /root/mlflow/artifacts --artifacts-destination /root/mlflow/artifacts"]
