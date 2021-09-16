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

### Other

* You can delete `access_token` file if there is an unknown error.
* `processed_invoices.json` is the file that tracks if the invoice was pulled and sent off
* `logs/*` is created and will have debug information
* You can goto the Xero Developer URL. Under history, it shows prior API calls for troubleshooting