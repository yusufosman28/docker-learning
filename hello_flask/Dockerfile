




 # Stage 1: Build stage
 FROM python:3.12-slim AS build 

 WORKDIR /app

 RUN apt-get update && apt-get install -y gcc pkg-config default-libmysqlclient-dev
 RUN pip install flask mysqlclient

 COPY . .

 
 # Stage 2: Production stage

 FROM python:3.12-slim 

 WORKDIR /app

 RUN apt-get update && apt-get install -y libmariadb3

 COPY --from=build  /app /app
 COPY --from=build /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages

 EXPOSE 5003

 CMD ["python", "app.py"]



