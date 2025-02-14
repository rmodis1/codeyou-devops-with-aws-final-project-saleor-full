# Establish base image
FROM python:3.12

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    APP_HOME=/app \
    DATABASE_URL=postgres://saleor:saleor@db:5432/saleor \
    ALLOWED_HOSTS=localhost,api,dashboard,0.0.0.0 \
    ALLOWED_CLIENT_HOSTS=dashboard \
    JAEGER_AGENT_HOST=jaeger

# Set work directory
WORKDIR $APP_HOME

# Install system dependencies
RUN apt-get update \
    && apt-get install -y \
        gettext \
        libffi8 \
        liblcms2-2 \
        libopenjp2-7 \
        libssl3 \
        libtiff6 \
        libwebp7 \
        shared-mime-info \
        mime-support \
        libpq5 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Poetry
ENV POETRY_VERSION=1.8.4
RUN pip install --no-cache-dir poetry==$POETRY_VERSION
COPY pyproject.toml poetry.lock /app/
RUN poetry config virtualenvs.create false \
    && poetry install --no-root --no-interaction --no-ansi

# Create non-root user
RUN useradd -m -d /app saleor \
    && chown -R saleor:saleor /app

# Copy application code
COPY . .

# Configure static files
ARG STATIC_URL
ARG SECRET_KEY
ENV STATIC_URL=${STATIC_URL:-/static/}
RUN SECRET_KEY=${SECRET_KEY:-dummy} STATIC_URL=${STATIC_URL} python3 manage.py collectstatic --no-input

# Switch to non-root user
USER saleor

# Expose port
EXPOSE 8000

# Start command
CMD ["gunicorn", "--bind", ":8000", "--workers", "4", "--worker-class", "saleor.asgi.gunicorn_worker.UvicornWorker", "saleor.asgi:application"]
