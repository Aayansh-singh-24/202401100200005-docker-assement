# Stage 3+ secure backend image. The version can be set when building a release.
FROM python:3.13.15-slim-trixie

ARG APP_VERSION=dev
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    APP_VERSION=${APP_VERSION}

WORKDIR /app

COPY requirements.txt ./
RUN python -m pip install --no-cache-dir --upgrade pip && \
    python -m pip install --no-cache-dir -r requirements.txt

RUN groupadd --system app && useradd --system --gid app --create-home app
COPY --chown=app:app app.py ./app.py
USER app

EXPOSE 5000
CMD ["python", "app.py"]
