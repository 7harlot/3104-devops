# Lab 11 Part I: build our own image for a tiny Flask app.

# Base image: a slim Debian with Python 3.11 already installed.
# This is why your PC never needs Python for this lab.
FROM python:3.11-slim

# Every command below runs inside /app in the image.
WORKDIR /app

# Copy the build context (this folder) into /app.
# .dockerignore keeps junk like screenshots out of the image.
COPY . /app

# Install Flask + Redis client listed in requirements.txt.
# --no-cache-dir keeps the image smaller (pip's download cache is useless in an image).
RUN pip install --no-cache-dir --trusted-host pypi.python.org -r requirements.txt

# Documentation only: the app listens on port 80 inside the container.
# The real port mapping happens with `docker run -p 4000:80`.
EXPOSE 80

# app.py reads NAME and prints "Hello <NAME>!".
# key="value" form; the doc's `ENV NAME World` form triggers a LegacyKeyValueFormat warning.
ENV NAME="Ben"

# What runs when a container starts from this image.
CMD ["python", "app.py"]
