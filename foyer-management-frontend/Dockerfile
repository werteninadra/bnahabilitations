# Étape 1 : Builder Angular avec Node 20
FROM node:20 AS build
WORKDIR /app

# Copier les fichiers package.json et package-lock.json
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste du code
COPY . .

# Builder Angular en mode production
RUN npm run build --prod

# Étape 2 : Servir via Nginx
FROM nginx:alpine

# Copier les fichiers construits dans Nginx
COPY --from=build /app/dist/* /usr/share/nginx/html

# Copier la configuration Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Exposer le port HTTP
EXPOSE 80

# Lancer Nginx
CMD ["nginx", "-g", "daemon off;"]