# Setting up zipline

This project provides a simple way to generate secure environment secrets using OpenSSL and save them to a `.env` file.

## Usage

Run the following commands in your terminal to generate a `.env` file with a PostgreSQL password and a core application secret:

````bash
echo "POSTGRESQL_PASSWORD=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" > .env
echo "CORE_SECRET=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" >> .env
