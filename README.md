This is a mailing microservice which uses external services to actually sent the emails. 
When such an external service is unavailable there is a fallback to a secondary service. 

The tech stack is lumen, docker, mysql, nginx.

Installation steps:
1. Clone the project
2. 'cd' to the project and run 'docker-compose build'
3. copy .env.example to .env and enter the data as follow:

DB_CONNECTION=mysql
DB_HOST=<mailservice_mysql-service-ip>
DB_PORT=3306
DB_DATABASE=mailservice
DB_USERNAME=root
DB_PASSWORD=root

CACHE_DRIVER=file
QUEUE_CONNECTION=sync

MAILJET_APIKEY=your_mailjet_apikey
MAILJET_APISECRET=your_mailjetapisecret_key

SENDGRID_API_KEY=sendgrid_apikey

RABBITMQ_HOST=<mailservice_rabbitmq-service-ip>

4. run 'docker-compose up'
 - In case mysql exits with exit code 1 run 'docker-compose up mysql'
5. Obtain the IP of the mysql service by executing 'docker inspect mailservice_mysql'
 - It is near the bottom of the output looking similar to:
 "IPAddress": "192.168.160.4"
6. put the IP in the laravel project .env file as DB_HOST value:
    DB_HOST=192.168.160.4
7. run 'docker-compose exec app php artisan migrate'

Below is an example JSON payload:
```javascript
{
  "from": {
    "email": "petar.ivanov2001@mail.bg",
    "name": "Petar"
  },
  "to": {
    {
      "email": "petar.ivanov2001@mail.bg",
      "name": "Petar"
    }
  },
  "subject": "Greetings from Mailjet.",
  "text": "My first Mailjet email",
  "html": "<h3>Dear passenger 1, welcome to <a href='https://www.mailjet.com/'>Mailjet</a>!</h3><br />May the delivery force be with you!",
}
```
You can use postman or curl or whatever tool you like to send the json payload to the api.
send it to the following endpoint:
http://<IP-address>/sendmail
 - to get the endpoint ip address run 'docker inspect mailservice_nginx'
 - test with curl:
 curl -X POST -H "Content-Type: application/json" \
 -d '{"from":{"email":"petar.ivanov2001@mail.bg","name":"Petar"},"to":{"email":"petar.ivanov2001@mail.bg","name":"Petar"},"subject":"Greetings from Mailjet.","text":"My first Mailjet email","html":"<h3>Dear passenger 1, welcome to <a href='https://www.mailjet.com/'>Mailjet</a>!</h3><br />May the delivery force be with you!"}' \
 http://<IP-address>/sendmail

Now the emails should be added to the queue.
To consume them from the rabbitmq queue run the following command:
docker-compose exec app php artisan consume:email

Checking the data in the db:
1. docker exec -it mailservice_mysql /bin/bash
2. mysql -u root -p
 - you will be prompted to enter a password - enter root
3. select * from mailservice.emails;

The publisher and worker log the information in the log files with the same names located in storage/logs.
