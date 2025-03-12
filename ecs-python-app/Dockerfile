# Use the official Python image
FROM python:3.9

# Set the working directory inside the container
WORKDIR /app

# Copy application files
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

# Expose port 5000
EXPOSE 5000

# Run the application
CMD ["python", "app.py"]