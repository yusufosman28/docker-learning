FROM python:3.12-slim
WORKDIR /app
COPY . .
RUN apt-get update && apt-get install -y gcc pkg-config default-libmysqlclient-dev
RUN pip install flask mysqlclient
EXPOSE 5003
CMD ["python", "app.py"]
