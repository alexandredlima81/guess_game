# Dockerfile para o Backend (Flask)
FROM python:3.10-slim

# Define o diretório de trabalho
WORKDIR /app

# Copia os arquivos de requisitos e instala as dependências
COPY ../requirements.txt ../requirements-dev.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copia o código da aplicação
COPY ./guess ./guess
COPY ../repository ./repository
COPY ../run.py ./

# Expõe a porta que o Flask usará
EXPOSE 5000

# Define as variáveis de ambiente para o Flask
ENV FLASK_APP=run.py
ENV FLASK_DB_TYPE=postgres
ENV FLASK_DB_USER=postgres
ENV FLASK_DB_NAME=postgres
ENV FLASK_DB_PASSWORD=secretpass
ENV FLASK_DB_HOST=db
ENV FLASK_DB_PORT=5432

# Comando para iniciar a aplicação Flask
CMD ["flask", "run", "--host=0.0.0.0"]
