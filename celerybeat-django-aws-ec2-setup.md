# CeleryBeat Setup for Django on AWS EC2

**Step 1: Create Celery Beat Configuration File**

    sudo nano /etc/default/celerybeat-your_project_name


Add the following content to the file and save:
Replace your_user with your users name
Replace your_virtualenv with path to your virtualenv 
Replace your_django_app with your django app name where celery.py exists

    CELERY_BIN="/home/your_user/your_virtualenv/bin/celery"
    CELERY_APP="your_django_app.celery:app"
    CELERYBEAT_PID_FILE="/var/run/celery/beat.pid"
    CELERYBEAT_LOG_FILE="/var/log/celery/beat.log"
    CELERYBEAT_LOG_LEVEL="INFO"



**Step 2: Create Celery Beat Systemd Service Unit**

    sudo nano /etc/systemd/system/celerybeat-your_project_name.service


Add the following content to the file and save:
Replace your_user with your users name
Replace your_django_project with your django project directory name.


    [Unit]
    Description=Celery Beat Service
    After=network.target

    [Service]
    Type=simple
    User=your_user
    Group=www-data
    EnvironmentFile=/etc/default/celerybeat-your_project_name
    WorkingDirectory=/home/your_user/your_django_project/
    ExecStart=/bin/sh -c '${CELERY_BIN} -A ${CELERY_APP} beat \
      --pidfile=${CELERYBEAT_PID_FILE} \
      --logfile=${CELERYBEAT_LOG_FILE} \
      --loglevel=${CELERYBEAT_LOG_LEVEL}'
    Restart=always
    RestartSec=5

    [Install]
    WantedBy=multi-user.target



**Step 3: Restart Celery Beat**

    sudo systemctl daemon-reload

    sudo systemctl enable celerybeat-your_project_name

    sudo systemctl start celerybeat-your_project_name

    sudo systemctl status celerybeat-your_project_name


    
