FROM selenium/standalone-chrome:122.0

ENV DISPLAY=':20'

# Switch to root user to create non-root user and install packages
USER root

# Create a non-root user and group
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

# Install necessary packages as root
RUN apt-get update && apt-get install -y ftp x11vnc ffmpeg libvpx7 && \
    curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g typescript && \
    node -v
    
# Ensure the non-root user can write to the supervisor log directory
RUN mkdir -p /var/log/supervisor && \
    chown -R appuser:appgroup /var/log/supervisor

# Copy entrypoint script and set permissions
COPY --chown=appuser:appgroup build/dockerfiles/entrypoint.sh /tmp/
RUN chmod +x /tmp/entrypoint.sh

# Create e2e directory and set permissions
RUN mkdir /tmp/e2e && \
    chown -R appuser:appgroup /tmp/e2e && \
    chmod -R 775 /tmp/e2e

# Modify supervisord configuration
RUN sed -i "s/nodaemon=true/nodaemon=false/" /etc/supervisord.conf

# Copy package files and install dependencies
COPY --chown=appuser:appgroup package.json package-lock.json /tmp/e2e/
RUN cd /tmp/e2e && \
    npm install

# Copy the rest of the application files
COPY --chown=appuser:appgroup . /tmp/e2e

# Set the working directory
WORKDIR /tmp/e2e

# Expose necessary port
EXPOSE 5920

# Switch to the non-root user
USER appuser

# Set entrypoint
ENTRYPOINT [ "/tmp/entrypoint.sh" ]