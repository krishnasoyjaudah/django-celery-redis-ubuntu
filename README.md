# django-celery-redis-ubuntu
Steps to deploy django celery redis on ubuntu 22.04  

Assuming that you have already installed celery and redis on your server, follow the instructions below.

Make sure your virtual environment is activated on your server.

Install Redis Server  
`sudo apt install redis-server`  
  
Start Redis Server  
`sudo systemctl start redis-server`  
  
Check Redis Server Status  
`sudo systemctl status redis`  

Test Redis client  
`redis-cli ping`  
`You should get response PONG`  

Redis conf  
`sudo nano /etc/redis/redis.conf`  
You can bind IP and set password (#requirepass line)

Install supervisor  
`sudo apt install supervisor`  
  
2 supervisor conf files needed: `celery.conf` and `celerybeat.conf`  

celery.conf  `This must be in folder /etc/supervisor/conf.d`  
```  
[program:celery]
command=/home/<user>/pyapps/venv/bin/celery -A <folder that contains celery.py file (Usually the same folder as settings.py)> worker --loglevel=INFO --hostname=<SERVER IP>
directory=/home/<user>/pyapps/<project folder>/
user=<user> # or you can use root but not recommended
numprocs=1
stdout_logfile=/var/log/supervisor/celery.log
stderr_logfile=/var/log/supervisor/celery.log
autostart=true
autorestart=true
startsecs=0

; Need to wait for currently executing tasks to finish at shutdown.
; Increase this if you have very long running tasks.
stopwaitsecs = 600

; When resorting to send SIGKILL to the program to terminate it
; send SIGKILL to its whole process group instead,
; taking care of its children as well.
killasgroup=true

; if rabbitmq is supervised, set its priority higher
; so it starts first
priority=998
```

  
celerybeat.conf  `This must be in folder /etc/supervisor/conf.d`
```
; ========================
;  celery beat supervisor
; ========================


; the name of your supervisord program
[program:celerybeat]

; Set full path to celery program if using virtualenv
command=/home/djangoadmin/pyapps/venv/bin/celery -A dear beat --loglevel=INFO

; The directory to your Django project
directory=/home/<user>/pyapps/<project folder>/
user=<user> # or you can use root but not recommended

; Supervisor will start as many instances of this program as named by numprocs
numprocs=1

; Put process stdout output in this file
stdout_logfile=/var/log/supervisor/celerybeat.log

; Put process stderr output in this file
stderr_logfile=/var/log/supervisor/celerybeat.log

; If true, this program will start automatically when supervisord is started
autostart=true

; May be one of false, unexpected, or true. If false, the process will never
; be autorestarted. If unexpected, the process will be restart when the program
; exits with an exit code that is not one of the exit codes associated with this
; process’ configuration (see exitcodes). If true, the process will be
; unconditionally restarted when it exits, without regard to its exit code.
autorestart=true

; The total number of seconds which the program needs to stay running after
; a startup to consider the start successful.
startsecs=0

; if your broker is supervised, set its priority higher
; so it starts first
priority=999
```  
  
In settings.py, change 127.0.0.1 to localhost in CELERY_BROKER_URL  
```
# CELERY SETTINGS
CELERY_BROKER_URL  = 'redis://localhost:6379/0'
CELERY_ACCEPT_CONTENT = ['application/json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
CELERY_TIMEZONE = TIME_ZONE or 'UTC'


CELERY_RESULT_BACKEND  = 'django-db' # I use django-celery-results


# CELERY BEAT SETTINGS
CELERY_BEAT_SCHEDULER = 'django_celery_beat.schedulers:DatabaseScheduler'
```


Restart supervisord  
`sudo service supervisor restart`

Reread & Update config files  
`sudo supervisorctl reread`  
`sudo supervisorctl update`

Start & Stop Processes  
`sudo supervisorctl status <process_name>`  
`sudo supervisorctl start <process_name>`  
`sudo supervisorctl stop <process_name>`  
`sudo supervisorctl restart <process_name>`  
`sudo supervisorctl start all`  
`sudo supervisorctl stop all`  
