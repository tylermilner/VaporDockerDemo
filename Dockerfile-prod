# This Dockerfile uses Docker multi-stage builds to create a production Docker image for a Vapor server-side Swift application.

#
# First stage - build image
#

# Start with the official Swift image
FROM swift:4.2 as builder

# Set the working directory to /app
WORKDIR /app

# Copy everything from the current project directory to the image
COPY . .

# Create a temporary "build" directory and copy the Swift static libraries into it
RUN mkdir -p /build/lib && cp -R /usr/lib/swift/linux/*.so /build/lib

# Create a release build of the app and copy it into the build directory
RUN swift build -c release && mv `swift build -c release --show-bin-path` /build/bin

#
# Second stage - production image
#

# Start with the official Ubuntu image
FROM ubuntu:16.04

# Install additional Swift runtime dependencies and then cleanup apt-get
RUN apt-get -qq update && apt-get install -y \
  libicu55 libxml2 libbsd0 libcurl3 libatomic1 \
  && rm -r /var/lib/apt/lists/*

# Set the working directory to /app
WORKDIR /app

# Copy the app binary into the production image
COPY --from=builder /build/bin/Run .

# Copy the Swift static libraries into the production image
COPY --from=builder /build/lib/* /usr/lib/

# Expose port 8080 to other containers. This also serves as documentation that port 8080 should be published when the container is run.
EXPOSE 8080

# Define the app as the entrypoint of the container
ENTRYPOINT ./Run serve -e prod -b 0.0.0.0
