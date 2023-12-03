# Celery Setup for Django on AWS EC2

This guide walks you through the process of setting up Celery for a Django application on an AWS EC2 instance running Ubuntu. Celery is a distributed task queue that helps you execute tasks asynchronously, making it a useful tool for handling background processes.

**Step 1: Create /etc/default/celeryd directory if it doesn't exist**


    # Create the directory if it doesn't exist
    sudo mkdir -p /etc/default/celeryd

**Step 2: Create the Celery configuration file**


    # Open the configuration file for editing
    sudo nano /etc/default/celeryd/your_project_name


    # You can give any name for your_project_name. Dont need any relation with django project name.

Add the following content to the file and save:
Replace your_user with your users name
Replace your_virtualenv with path to your virtualenv 
Replace your_django_app with your django app name where celery.py exists


    # Name of nodes to start
    CELERYD_NODES="w1"

    # Absolute or relative path to the 'celery' command:
    CELERY_BIN="/home/your_user/your_virtualenv/bin/celery"

    # App instance to use
    CELERY_APP="your_django_app.celery:app"

    # How to call manage.py
    CELERYD_MULTI="multi"

    # Extra command-line arguments to the worker
    CELERYD_OPTS="--time-limit=300 --concurrency=2"

    # PID and log files
    CELERYD_PID_FILE="/var/run/celery/%n.pid"
    CELERYD_LOG_FILE="/var/log/celery/%n%I.log"
    CELERYD_LOG_LEVEL="INFO"

    # Celery Beat options
    CELERYBEAT_PID_FILE="/var/run/celery/beat.pid"
    CELERYBEAT_LOG_FILE="/var/log/celery/beat.log"


**Step 3: Create the systemd service file**


    # Open the systemd service file for editing
    sudo nano /etc/systemd/system/celery-your_project_name.service


Add the following content to the file and save:
Replace your_user with your users name
Replace your_django_project with your django project directory name.


    [Unit]
    Description=Celery Service
    After=network.target

    [Service]
    Type=forking
    User=your_user
    Group=www-data
    EnvironmentFile=/etc/default/celeryd/your_project_name
    WorkingDirectory=/home/your_user/your_django_project/
    ExecStart=/bin/sh -c '${CELERY_BIN} multi start ${CELERYD_NODES} \
      -A ${CELERY_APP} --pidfile=${CELERYD_PID_FILE} \
      --logfile=${CELERYD_LOG_FILE} --loglevel=${CELERYD_LOG_LEVEL} ${CELERYD_OPTS}'
    ExecStop=/bin/sh -c '${CELERY_BIN} multi stopwait ${CELERYD_NODES} \
      --pidfile=${CELERYD_PID_FILE}'
    ExecReload=/bin/sh -c '${CELERY_BIN} multi restart ${CELERYD_NODES} \
      -A ${CELERY_APP} --pidfile=${CELERYD_PID_FILE} \
      --logfile=${CELERYD_LOG_FILE} --loglevel=${CELERYD_LOG_LEVEL} ${CELERYD_OPTS}'

    [Install]
    WantedBy=multi-user.target


**Step 4: Create the log directory**


    # Create the log directory
    sudo mkdir -p /var/log/celery


**Step 5: Adjust ownership of the directory**


    # Adjust ownership of the log directory
    sudo chown -R your_user:your_group /var/log/celery

    #Replace your_user and your_group with the appropriate values. You can find your user and group with the id command:
    id


**Step 6: Restart the server**


    # Reload systemd configuration
    sudo systemctl daemon-reload

    # Enable the Celery service
    sudo systemctl enable celery-your_project_name

    # Restart the Celery service
    sudo systemctl restart celery-your_project_name

    # Check the status of the Celery service
    sudo systemctl status celery-your_project_name


    
