# Dockerfile

# Choose a Python base image that includes build tools
FROM python:3.10-slim-bullseye as builder

# Set environment variables to prevent interactive prompts during build
ENV PYTHONUNBUFFERED=1 \
    DEBIAN_FRONTEND=noninteractive

# Install essential build tools and wget
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    wget \
    && rm -rf /var/lib/apt/lists/*

# --- TA-Lib Installation ---
# 1. Download TA-Lib C library source
WORKDIR /tmp
RUN wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz && \
    tar -xzf ta-lib-0.4.0-src.tar.gz && \
    rm ta-lib-0.4.0-src.tar.gz

# 2. Build and install the TA-Lib C library
WORKDIR /tmp/ta-lib
RUN ./configure --prefix=/usr && \
    make && \
    make install

# --- Python Dependencies ---
# Use a new stage for a cleaner final image
FROM python:3.10-slim-bullseye

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    DEBIAN_FRONTEND=noninteractive

# Copy TA-Lib C library files from the builder stage
COPY --from=builder /usr/lib/libta_lib* /usr/lib/
COPY --from=builder /usr/include/ta-lib/* /usr/include/

# Install runtime dependencies (like wget if needed, but unlikely now)
# RUN apt-get update && apt-get install -y --no-install-recommends ... && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements file first for layer caching
COPY requirements.txt .

# Install Python dependencies (including the TA-Lib Python wrapper)
# Use --no-cache-dir to reduce image size
RUN pip install --no-cache-dir -r requirements.txt

# --- Application Code ---
# Copy your bot script, the api folder, and ML models
COPY your_bot_script.py .
COPY api/ ./api/
COPY *.joblib ./

# --- Run Command ---
# Command to start your bot
CMD ["python", "app.py"]
