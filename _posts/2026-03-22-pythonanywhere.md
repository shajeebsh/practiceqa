---
title: "PythonAnywhere"
date: 2026-03-22 17:15:49 
author: shajeebhameed
layout: page
slug: pythonanywhere
status: publish
---

# Deploying Olive ERP on PythonAnywhere: Complete Guide

## Overview

This guide walks you through deploying the Olive ERP system (Django 4.2 + Wagtail CMS) on PythonAnywhere. The application includes modules for finance, inventory, CRM, HR, project management, and compliance tracking.

* * *

## Prerequisites

  * A GitHub account
  * A PythonAnywhere account
  * Basic familiarity with command line operations



* * *

## Part 1: PythonAnywhere Account Setup

### Step 1.1: Create Account and Choose Plan

  1. Navigate to **[www.pythonanywhere.com](<http://www.pythonanywhere.com/>)** and sign up
  2. Select a plan based on your needs:

Plan| Price| MySQL| External DB Access| Always-on Tasks  
---|---|---|---|---  
Free (Beginner)| $0| ❌ No| ❌ No| ❌ No  
Hacker| $5/month| ✅ Yes| ✅ Yes| ❌ No  
Web Developer| $12/month| ✅ Yes| ✅ Yes| ✅ Yes  
  
**Important Considerations:**

  * Free tier does **not** include MySQL or external database access
  * Celery workers require always-on tasks (Web Developer plan)
  * For production ERP, the Web Developer plan is recommended



* * *

## Part 2: Python Environment Setup

### Step 2.1: Create Virtual Environment

Open a Bash console in PythonAnywhere and run:

bash

# Create virtual environment with Python 3.10

mkvirtualenv olive_erp --python=/usr/bin/python3.10

# Activate the environment

workon olive_erp

# Verify Python version

python --version

**Troubleshooting:** If you encounter `_posixsubprocess` errors, delete and recreate the virtual environment:

bash

# Remove broken environment

rm-rf ~/.virtualenvs/olive_erp

# Recreate with Python 3.10

mkvirtualenv olive_erp --python=/usr/bin/python3.10

* * *

## Part 3: Clone and Configure Application

### Step 3.1: Clone the Repository

bash

# Navigate to home directory

cd ~

# Clone the repository

git clone https://github.com/shajeebsh/olive_erp.git

# Enter the project directory

cd olive_erp

### Step 3.2: Install Dependencies

bash

# Ensure virtual environment is active

workon olive_erp

# Upgrade pip (use python -m pip for reliability)

python -m pip install --upgrade pip

# Install required packages

pip install dj-database-url

pip install django-celery-beat

pip install psycopg2-binary

pip install python-dotenv

# Install from requirements.txt

pip install -r requirements.txt

* * *

## Part 4: Database Configuration

### Option A: PostgreSQL with Neon (Requires Paid Plan)

**Step 4A.1: Create Neon Database**

  1. Go to **neon.tech** and sign up
  2. Create a new project
  3. Name your database (e.g., `oliveerp_db`)
  4. Copy the connection string provided



**Step 4A.2: Configure Environment**

Create `.env.prod` file:

bash

cd ~/olive_erp

nano .env.prod

Add the following content:

env

# Production Environment Variables

DEBUG=False

# Generate with: python -c "import secrets; print(secrets.token_urlsafe(50))"

SECRET_KEY=your-generated-secret-key-here

# PostgreSQL (Neon)

DATABASE_URL=postgresql://username:password@ep-xxx.us-east-1.aws.neon.tech/dbname?sslmode=require

# Redis (Upstash - for Celery)

REDIS_URL=redis://default:password@host.upstash.io:6379

# Production Hosts

ALLOWED_HOSTS=yourusername.pythonanywhere.com,127.0.0.1

### Option B: SQLite (Free Tier Workaround)

For the free tier, use SQLite as the database:

bash

cd ~/olive_erp

nano .env.prod

Add the following content:

env

# Production Environment Variables

DEBUG=False

# Generate with: python -c "import secrets; print(secrets.token_urlsafe(50))"

SECRET_KEY=your-generated-secret-key-here

# SQLite Database (for free tier)

DATABASE_URL=sqlite:///home/yourusername/olive_erp/db.sqlite3

Eg:- MySQL (Local) - DATABASE_URL=sqlite:////home/oliveerp/olive_erp/db.sqlite3

# Production Hosts

ALLOWED_HOSTS=yourusername.pythonanywhere.com,127.0.0.1

**Note:** SQLite is suitable for development and testing only. For production ERP systems, use PostgreSQL or MySQL.

* * *

## Part 5: Generate Secret Key

Run this command to generate a secure secret key:

bash

python -c"import secrets; print(secrets.token_urlsafe(50))"

Copy the output and paste it into your `.env.prod` file as the `SECRET_KEY` value.

* * *

## Part 6: Update Django Settings

### Step 6.1: Update Celery Configuration

Edit the base settings file:

bash

nano ~/olive_erp/wagtailerp/settings/base.py

Find the Celery settings section and update to use environment variables:

python

# Before (hardcoded):

# CELERY_BROKER_URL = 'redis://localhost:6379/0'

# CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'

# After (using environment variables):

CELERY_BROKER_URL=config('REDIS_URL', default='redis://localhost:6379/0')

CELERY_RESULT_BACKEND=config('REDIS_URL', default='redis://localhost:6379/0')

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

* * *

## Part 7: Initialize Database

### Step 7.1: Run Migrations

bash

workon olive_erp

cd ~/olive_erp

# Run database migrations

python manage.py migrate

# Create admin user

python manage.py createsuperuser

# Collect static files

python manage.py collectstatic #--noinput

### Step 7.2: Create Media Directory

bash

mkdir-p ~/olive_erp/media

* * *

## Part 8: Configure Web Application

### Step 8.1: Create Web App

  1. Go to the **Web** tab in PythonAnywhere dashboard
  2. Click **"Add a new web app"**
  3. Choose your domain: `yourusername.pythonanywhere.com`
  4. Select **"Manual configuration"**
  5. Choose **Python 3.10**



### Step 8.2: Configure WSGI File

In the Web tab, click on the WSGI file link and replace the content with:

import os  
import sys

# Add project directory to path

project_home = '/home/yourusername/olive_erp'  
if project_home not in sys.path:  
sys.path.insert(0, project_home)

# Set Django settings module

os.environ['DJANGO_SETTINGS_MODULE'] = 'wagtailerp.settings'

# Load environment variables from .env

from dotenv import load_dotenv  
load_dotenv(os.path.join(project_home, '.env'))

# Import Django WSGI application

from django.core.wsgi import get_wsgi_application  
application = get_wsgi_application()

**Important:** Replace `yourusername` with your actual PythonAnywhere username.

### Step 8.3: Set Virtual Environment

In the **Virtualenv** section of the Web tab, enter:

text

/home/yourusername/.virtualenvs/olive_erp

### Step 8.4: Set Working Directory

In the **Working directory** field, enter:

text

/home/yourusername/olive_erp

### Step 8.5: Configure Static Files

In the **Static files** section, add these mappings:

URL| Directory  
---|---  
`/static/`| `/home/yourusername/olive_erp/staticfiles/`  
`/media/`| `/home/yourusername/olive_erp/media/`  
  
* * *

## Part 9: Redis Setup for Celery (Optional)

### Step 9.1: Create Upstash Redis Account

  1. Go to **upstash.com** and sign up (free tier available)
  2. Create a new Redis database
  3. Copy the connection URL



### Step 9.2: Add Redis URL to Environment

Add to your `.env.prod` file:

env

REDIS_URL=redis://default:your-password@your-host.upstash.io:6379

### Step 9.3: Configure Always-On Task (Paid Plans)

For paid plans with always-on tasks:

  1. Go to the **Tasks** tab
  2. Create a new **Always-on task**
  3. Enter the command:



bash

cd /home/yourusername/olive_erp && /home/yourusername/.virtualenvs/olive_erp/bin/celery -A wagtailerp worker --loglevel=info

* * *

## Part 10: Final Deployment

### Step 10.1: Reload Web Application

Click the **Reload** button in the Web tab.

### Step 10.2: Test Your Deployment

Visit these URLs to verify:

URL| Purpose  
---|---  
`https://yourusername.pythonanywhere.com/`| Main site  
`https://yourusername.pythonanywhere.com/admin/`| Django admin  
`https://yourusername.pythonanywhere.com/cms/`| Wagtail CMS  
  
* * *

## Troubleshooting

### Common Errors and Solutions

Error| Solution  
---|---  
`ModuleNotFoundError: No module named 'dj_database_url'`| `pip install dj-database-url`  
`ModuleNotFoundError: No module named 'django_celery_beat'`| `pip install django-celery-beat`  
`ModuleNotFoundError: No module named 'psycopg2'`| `pip install psycopg2-binary`  
`ModuleNotFoundError: No module named '_posixsubprocess'`| Delete and recreate virtual environment  
`Connection refused` (PostgreSQL)| Upgrade to paid plan for external DB access  
`502 Bad Gateway`| Check WSGI file, virtualenv path, and error logs  
`Static files 404`| Run `collectstatic`, verify static files mapping  
  
### Checking Logs

bash

# View server log

tail -f /var/log/yourusername.server.log

# View access log

tail -f /var/log/yourusername.access.log

* * *

## Quick Reference Commands

bash

# Activate virtual environment

workon olive_erp

# Navigate to project

cd ~/olive_erp

# Pull latest changes

git pull origin master

# Install dependencies

pip install -r requirements.txt

# Run migrations

python manage.py migrate

# Collect static files

python manage.py collectstatic #--noinput

# Check for deployment issues

python manage.py check --deploy

# Generate secret key

python -c"import secrets; print(secrets.token_urlsafe(50))"

* * *

## Deployment Checklist

Step| Description| Status  
---|---|---  
1| Create PythonAnywhere account| ☐  
2| Create virtual environment| ☐  
3| Clone repository| ☐  
4| Install dependencies| ☐  
5| Set up database (Neon or SQLite)| ☐  
6| Create `.env.prod` file| ☐  
7| Generate SECRET_KEY| ☐  
8| Update Celery settings| ☐  
9| Run migrations| ☐  
10| Create superuser| ☐  
11| Collect static files| ☐  
12| Configure web app| ☐  
13| Set up WSGI file| ☐  
14| Configure static file mappings| ☐  
15| Set up Redis (optional)| ☐  
16| Reload and test| ☐  
  
* * *

## Summary

This guide covers the complete deployment process for Olive ERP on PythonAnywhere. Key considerations include:

  1. **Plan Selection:** Free tier has limitations; paid plans unlock external databases and always-on tasks
  2. **Database Options:** PostgreSQL (Neon) for production, SQLite for development on free tier
  3. **Background Tasks:** Celery requires Redis and always-on tasks (Web Developer plan)
  4. **Security:** Always generate a new SECRET_KEY for production



For production ERP deployments, the Web Developer plan ($12/month) is recommended as it provides all necessary features including MySQL/PostgreSQL access, always-on tasks for Celery workers, and custom domain support.
