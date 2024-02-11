FROM python
WORKDIR /app
COPY . .
# ENV MYSQL_DATABASE=firststep
# ENV MYSQL_ROOT_PASSWORD=passwd 
RUN pip install -r requirements.txt
CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]