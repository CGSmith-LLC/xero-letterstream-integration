# Xero to LetterStream
Xero is a platform for invoicing customers. [LetterStream](https://www.letterstream.com/) is
a platform that allows sending real mail via an API. This repository downloads invoice PDFs
and sends them to the LetterStream API so your customers receive a physical letter.

The script will only pull AUTHORISED invoices within the specified date range.

### Prerequisites

- Xero Account
- LetterStream Account
- Python 3.8
- Server to run Python on and receive OAuth calls
- [ngrok](https://ngrok.com) for tunnelling


### Setup

1. Copy `config.py.example` to `config.py`
   1. Change the `config.py` file for your own settings
   1. Xero 3rd party app and secret can be created here: https://developer.xero.com/myapps/
   1. In the app description, the "OAuth 2.0 redirect URI" field should be the same as the variables "appAddress + authSlug" in config.py
1. `pip3 install python-docx`
1. `export FLASK_APP=main.py`
1. `flask run --host=0.0.0.0`
1. In another terminal you can run `ngrok http 5000` to get an external address for testing

### URLs
Your URLs are configured in the config file. Navigate to the ngrok URL and goto `/auth` this will then redirect
to Xero to authorize your app. Once authorized you can then goto `/mailinvoice` and it will process
the days invoices.

Optionally you can pass `/runinvoice?startdate=yyyy-mm-dd&enddate=yyyy-mm-dd` to search for older invoices.

### Production run

To run this on your server install nginx and gunicorn. You can lookup running a flask app in production but here is our 
gunicorn systemd file:

```shell
[Unit]
Description=Gunicorn instance to serve xero_app
After=network.target

[Service]
User=xero_user
Group=www-data
WorkingDirectory=/home/xero_user/xero_app
Environment="PATH=/home/xero_user/xero_app/xero_env/bin"
ExecStart=/home/xero_user/xero_app/xero_env/bin/gunicorn --workers 3 --bind unix:xero_app.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

We also use [Buddy.Works](https://buddy.works) for deploying the application. Here is an example of what we run to automatically deploy changes.

```yaml
- pipeline: "Build, Deploy"
  on: "EVENT"
  events:
  - type: "PUSH"
    refs:
    - "refs/heads/master"
  priority: "NORMAL"
  target_site_url: "https://cgsmith.net/xero-url"
  fail_on_prepare_env_warning: true
  actions:
  - action: "Execute: pip install python-docx"
    type: "BUILD"
    working_directory: "/buddy/xero-letterstream-integration"
    docker_image_name: "library/python"
    docker_image_tag: "3.8.10"
    execute_commands:
    - "pip install python-docx"
    cached_dirs:
    - "/root/.cache/pip"
    volume_mappings:
    - "/:/buddy/xero-letterstream-integration"
    cache_base_image: true
    shell: "BASH"
  - action: "Upload files to cgsmith.net"
    type: "DIGITAL_OCEAN"
    input_type: "BUILD_ARTIFACTS"
    remote_path: "/home/xero_user/xero_app"
    login: "xerouser"
    host: "1.2.3.4"
    host_name: "cgsmith.net"
    port: "9222"
    authentication_mode: "WORKSPACE_KEY"
    integration_hash: "hiimhash"
  - action: "[cgsmith.net] Execute: service nginx restart"
    type: "SSH_COMMAND"
    login: "xerouser"
    host: "1.2.3.4"
    host_name: "cgsmith.net"
    port: "9222"
    authentication_mode: "WORKSPACE_KEY"
    commands:
    - "service xero_app restart"
    - "service nginx restart"
    run_as_script: true
    shell: "BASH"
```

### Other

* You can delete `access_token` file if there is an unknown error.
* `processed_invoices.json` is the file that tracks if the invoice was pulled and sent off
* `logs/*` is created and will have debug information
* You can goto the Xero Developer URL. Under history, it shows prior API calls for troubleshooting