# Books-Server
Create a web application with python + Flask + PostgreSQL and deploy on Heroku

# Install Postgre SQL
Following : https://tecadmin.net/install-postgresql-server-on-ubuntu/

# Step 1 – Enable PostgreSQL Apt Repository

sudo apt-get install wget ca-certificates

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

# Step 2 – Install PostgreSQL on Ubuntu

sudo apt-get update

sudo apt-get install postgresql postgresql-contrib

# Step 3 – Create User for PostgreSQL

Now configure PostgreSQL to make is accessible by your normal users. Change your_username with your actual user already created on your Ubuntu system.

postgres-# CREATE ROLE your_username WITH LOGIN CREATEDB ENCRYPTED PASSWORD 'your_password';

postgres-# \q

Then switch to the user account and run createdb command followed by the database name. This will create a database on PostgreSQL.

su - your_username

createdb my_db // repalce my_db with the username

After that connect to the PostgreSQL server. You will be logged in and get database prompt. To list all available databases use these commands.
psql

talha=> \list

To disconnect from PostgreSQL database command prompt just type below command and press enter. It will return you back to the Ubuntu command prompt.

\q



# Reference
https://medium.com/@dushan14/create-a-web-application-with-python-flask-postgresql-and-deploy-on-heroku-243d548335cc
