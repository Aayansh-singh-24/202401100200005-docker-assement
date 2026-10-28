# Stage 3+ secure backend image. For Stage 1, temporarily use the documented
# Python 3.9 base and the starter requirements, then return to this Dockerfile.
FROM python:3.13.15-slim-trixie

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

COPY requirements.txt ./
RUN python -m pip install --no-cache-dir --upgrade pip && \
    python -m pip install --no-cache-dir -r requirements.txt

RUN groupadd --system app && useradd --system --gid app --create-home app
COPY --chown=app:app app.py ./app.py
USER app

EXPOSE 5000
CMD ["python", "app.py"]
