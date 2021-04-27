### This project creates a Django project from scratch and deploy to AWS Elastic Beanstalk environment
### Local ops
- Install required packages
```bash
pip install virtualenv awsebcli
# pip install pillow psycopg2
```
- Create a working directory:
```bash
mkdir django
cd django
python -m venv eb-virt
./eb-virt/Scripts/activate
pip install django==2.2
django-admin startproject ebdjango
cd ebdjango
python manage.py runserver
pip freeze > requirements.txt
mkdir .ebextensions
touch .ebextensions/django.config
```
- Add following script to django.config file:
```bash
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: ebdjango.wsgi:application
```
- Deactivate virtual environment:
```bash
deactivate
```
### Configure Elastic Beanstalk using CLI locally
- Initialize an Elastic Beanstalk environment, needs installation and aws configuration first, than:
```bash
eb init -p python-3.7 django-tutorial   # python version 3.6 causes problems
eb init
eb create django-env
eb status
```
- Get the CNAME and paste it to ALLOWED_HOSTS
```py
ALLOWED_HOSTS = ['django-env.eba-pjyppy6b.us-west-2.elasticbeanstalk.com']
```
- Deploy, and check the final version:
```bash
eb deploy
eb open
```
### Create a site administrator
```bash
./eb-virt/Scripts/activate
cd .\ebdjango\
python manage.py migrate
python manage.py createsuperuser
```
- To tell Django where to store static files, define STATIC_ROOT in settings.py.
```py
STATIC_ROOT = 'static'
```
```bash
python manage.py collectstatic
eb deploy
eb open
```
- Open /admin dashboard, and check if login is possible or not.
### Add a database migration configuration file
- You can add commands to your .ebextensions script that are run when your site is updated. This enables you to automatically generate database migrations.
- Create a configuration file named db-migrate.config with the following content under .ebextensions folder.
```bash
container_commands:
  01_migrate:
    command: "django-admin.py migrate"
    leader_only: true
option_settings:
  aws:elasticbeanstalk:application:environment:
    DJANGO_SETTINGS_MODULE: ebdjango.settings
```
- This configuration file runs the django-admin.py migrate command during the deployment process, before starting your application. Because it runs before the application starts, you must also configure the DJANGO_SETTINGS_MODULE environment variable explicitly (usually wsgi.py takes care of this for you during startup). Specifying leader_only: true in the command ensures that it is run only once when you're deploying to multiple instances.
- Deploy and check.
```bash
eb deploy
```
### Security
- Create .env file under main project directory
- Add secret key:
```bash
SECRET_KEY=j^$&...
```
- Add .env to .gitignore
- Modify settings.py
```bash
from decouple import config
SECRET_KEY = config('SECRET_KEY')
from decouple import config
```
### Terminate the project
```bash
eb terminate django-env
rm -rf ~/eb-virt
rm -rf ~/ebdjango
```