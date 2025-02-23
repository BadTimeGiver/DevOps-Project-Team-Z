# Base image
FROM golang:alpine

# Set working directory
WORKDIR /app

# Copy source code
COPY . .

# Build application
RUN go build -o main .

# Expose the application port
EXPOSE 8081

# Start the application
CMD ["./main"]
