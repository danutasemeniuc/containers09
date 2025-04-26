# Containers09 - Optimizarea imaginilor Docker

## Scopul lucrării
Scopul acestei lucrări de laborator este înțelegerea procesului de optimizare a imaginilor Docker pentru a obține imagini de dimensiuni mai mici și mai eficiente.

## Sarcina
- Crearea unei imagini de bază pentru un site static folosind Nginx.
- Optimizarea succesivă a imaginii prin:
  - eliminarea dependențelor neutilizate,
  - reducerea numărului de straturi,
  - utilizarea unei imagini de bază minime (Alpine Linux),
  - repachetarea imaginii.
- Compararea dimensiunilor imaginilor rezultate.
- Pregătirea unui raport cu pașii realizați și concluziile.

## Pașii de realizare

### 1. Pregătirea mediului

Se creează un director numit `containers09`, în care se plasează:

- **Directorul `site/`** cu fișierele site-ului:

```plaintext
containers09/
├── site/
│   ├── index.html
│   ├── style.css
│   └── script.js
```

### 2. Conținutul fișierelor site-ului

**site/index.html**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Simple Site</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>This is a simple static page served by Nginx inside a Docker container.</p>
    <script src="script.js"></script>
</body>
</html>
```

**site/style.css**:
```css
body {
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
    text-align: center;
    margin-top: 50px;
}
```

**site/script.js**:
```javascript
console.log("Site loaded successfully!");
```

---

### 3. Crearea imaginii inițiale (mynginx:raw)

**Dockerfile.raw**:
```Dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Comenzi utilizate:
```bash
docker image build -t mynginx:raw -f Dockerfile.raw .
```

---

### 4. Eliminarea dependențelor neutilizate și fișierelor temporare (mynginx:clean)

**Dockerfile.clean**:
```Dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# remove apt cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Comenzi:
```bash
docker image build -t mynginx:clean -f Dockerfile.clean .
```

---

### 5. Minimizarea numărului de straturi (mynginx:few)

**Dockerfile.few**:
```Dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Comenzi:
```bash
docker image build -t mynginx:few -f Dockerfile.few .
```

---

### 6. Utilizarea unei imagini de bază minime (mynginx:alpine)

**Dockerfile.alpine**:
```Dockerfile
# create from alpine image
FROM alpine:latest

# update system
RUN apk update && apk upgrade

# install nginx
RUN apk add nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Comenzi:
```bash
docker image build -t mynginx:alpine -f Dockerfile.alpine .
```

---

### 7. Repachetarea imaginii (mynginx:repack)

Comenzi:
```bash
docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
```

---

### 8. Utilizarea tuturor metodelor (mynginx:minx și mynginx:min)

**Dockerfile.min**:
```Dockerfile
# create from alpine image
FROM alpine:latest

# update system, install nginx and clean
RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Comenzi:
```bash
docker image build -t mynginx:minx -f Dockerfile.min .
docker container create --name mynginx mynginx:minx
docker container export mynginx | docker image import - myngin:min
docker container rm mynginx
```

---

### 9. Compararea dimensiunilor imaginilor

Comandă:
```bash
docker image list
```

**Exemplu tabel rezultate** (valorile pot varia):

| Imagine         | Dimensiune  |
|-----------------|-------------|
| mynginx:raw     | 280 MB      |
| mynginx:clean   | 250 MB      |
| mynginx:few     | 245 MB      |
| mynginx:alpine  | 20 MB       |
| mynginx:repack  | 260 MB      |
| mynginx:minx    | 19 MB       |
| myngin:min      | 18 MB       |

---

## Răspunsuri la întrebări

### Care metodă de optimizare a imaginilor vi se pare cea mai eficientă?

**Cea mai eficientă metodă** este utilizarea unei **imagini de bază minime**, cum ar fi Alpine Linux, împreună cu **curățarea cache-ului** și **minimizarea numărului de straturi**. Aceasta reduce considerabil dimensiunea imaginii fără a compromite funcționalitatea.

### De ce curățirea cache-ului pachetelor într-un strat separat nu reduce dimensiunea imaginii?

Dacă curățarea cache-ului este efectuată într-un strat separat, straturile anterioare rămân în continuare stocate în imaginea Docker. Pentru reducerea reală a dimensiunii, curățarea trebuie făcută **în același RUN** cu instalarea, astfel încât stratul de cache să nu existe deloc.

### Ce este repachetarea imaginii?

Repachetarea imaginii constă în crearea unui container dintr-o imagine existentă, exportarea conținutului containerului ca arhivă și reimportarea arhivei ca o nouă imagine. Acest proces elimină metadatele straturilor intermediare, reducând astfel mărimea totală a imaginii.

---

## Concluzii

În cadrul acestei lucrări de laborator:
- Am construit și optimizat imagini Docker pentru un site static.
- Am demonstrat reducerea dimensiunii imaginilor prin utilizarea unor tehnici precum curățarea cache-ului, minimizarea numărului de straturi și folosirea unei imagini Alpine.
- Am învățat despre repachetarea imaginilor pentru eliminarea stratificării inutile.

Astfel, am aprofundat practicile de bună utilizare a Docker pentru crearea de imagini rapide, eficiente și ușor de utilizat în producție.

---
