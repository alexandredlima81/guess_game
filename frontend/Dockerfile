# Use a imagem oficial do Node.js
FROM node:14 AS builder

# Defina o diretório de trabalho
WORKDIR /app

# Copie os arquivos do package e instale as dependências
COPY package.json package-lock.json ./
RUN npm install

# Copie todo o restante do código-fonte do seu aplicativo
COPY . .

# Crie a build do aplicativo
RUN npm run build

# Use o nginx para servir o aplicativo
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


