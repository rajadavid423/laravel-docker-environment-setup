# Laravel Docker Environment Setup

This document explains the setup and step-by-step execution of a Laravel application using Docker. It includes the configuration of `docker-compose.yml` and `Dockerfile` files.

---

## **docker-compose.yml Explanation**

### **Key Sections**

#### **Version**
```yaml
version: "3.9"
```
Specifies the Docker Compose file format version.

#### **Services**

1. **`laravelapp`**
   - **Container Name**: `laravel-11`
   - **Build Context**: Uses the local `Dockerfile` to build the Laravel application image.
   - **Restart Policy**: Always restarts the container in case of failure.
   - **Ports**: Maps `8000` in the container to `8003` on the host.
   - **Environment File**: Loads environment variables from `.env`.
   - **Dependencies**: Depends on the `mysql` service.
   - **Network**: Connects to the `laravel` network.

2. **`mysql`**
   - **Container Name**: `mysql`
   - **Image**: `mysql:8.0`
   - **Ports**: Maps `3306` in the container to `3308` on the host.
   - **Authentication Plugin**: Uses `mysql_native_password`.
   - **Environment Variables**: Sets database credentials using `.env` variables.
   - **Volumes**: 
     - `./dump` for initializing the database.
     - `pgdata` for persisting database data.
     - `./conf` for custom MySQL configurations.
   - **Network**: Connects to the `laravel` network.

3. **`phpmyadmin`**
   - **Image**: `phpmyadmin`
   - **Ports**: Maps `80` in the container to `8081` on the host.
   - **Environment Variables**: Connects to `mysql` using `PMA_HOST=mysql`.
   - **Network**: Connects to the `laravel` network.

#### **Volumes**
```yaml
volumes:
  pgdata: {}
```
Defines a named volume to persist MySQL data.

#### **Networks**
```yaml
networks:
  laravel:
```
Creates a network to enable communication between services.

---

## **Dockerfile Explanation**

### **Key Instructions**

1. **Base Image**
```dockerfile
FROM php:8.2-apache
```
Uses PHP 8.2 with Apache as the web server.

2. **Install PHP Extensions**
```dockerfile
RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli
RUN docker-php-ext-install pdo pdo_mysql
RUN apt-get update && apt-get install -y \
    libpq-dev \
    nodejs \
    npm \
    git \
    unzip \
    && docker-php-ext-install pdo pdo_pgsql
```
Installs necessary PHP extensions for MySQL and PostgreSQL support, along with tools like Node.js and npm.

3. **Configure Apache**
```dockerfile
RUN a2enmod rewrite
RUN a2enmod ssl
RUN service apache2 restart
```
Enables `mod_rewrite` and `mod_ssl` for Laravel routing and HTTPS support.

4. **Install Composer**
```dockerfile
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
```
Installs Composer, a PHP dependency manager.

5. **Set Working Directory and Copy Application Code**
```dockerfile
WORKDIR /var/www/html
COPY . .
```
Sets the application root and copies code from the host to the container.

6. **Set Permissions**
```dockerfile
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache
```
Ensures Laravel's storage and cache directories are writable by the web server.

7. **Install Dependencies**
```dockerfile
RUN composer install
RUN npm install
```
Installs backend (PHP) and frontend (Node.js) dependencies.

8. **Add and Set Entrypoint Script**
```dockerfile
COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```
Copies and prepares a custom entrypoint script to initialize the application.

---

## **Step-by-Step Execution Order**

### **1. Start Docker Compose**
Run:
```bash
docker-compose up
```
Docker Compose reads the `docker-compose.yml` and performs the following:
- Builds the `laravelapp` service image using the `Dockerfile`.
- Starts the containers (`laravelapp`, `mysql`, `phpmyadmin`).

### **2. Build Process for `laravelapp`**
The `Dockerfile` is executed step by step:
1. **Base image setup**.
2. **Install PHP extensions**.
3. **Enable Apache modules**.
4. **Install Composer**.
5. **Set up application code**.
6. **Set permissions**.
7. **Install PHP and Node.js dependencies**.
8. **Add entrypoint script**.

### **3. Start `mysql` Service**
- Pulls and starts the MySQL 8.0 image.
- Sets up database credentials and initializes data from the `./dump` directory.

### **4. Start `phpmyadmin` Service**
- Pulls and starts the phpMyAdmin image.
- Connects to the `mysql` service.

### **5. Application Initialization**
- The `laravelapp` container runs the `entrypoint.sh` script to finalize initialization.

---

## **Accessing the Application**

- **Laravel Application**: Visit `http://localhost:8003`
- **phpMyAdmin**: Visit `http://localhost:8081`

---

This setup ensures a complete Laravel development environment with database management and service interconnection.
