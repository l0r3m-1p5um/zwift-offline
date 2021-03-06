# zoffline

zoffline enables the use of [Zwift](http://zwift.com) offline by acting as a partial implementation
of a Zwift server. Currently it's designed for only a single user and the UDP
game node is not implemented.

## Install

### Step 1: Install zoffline


<details><summary>Basic usage</summary>
To install zoffline on Linux, Windows, or Mac OS X:

* Install Python 2 (https://www.python.org/downloads/) if not already installed
* Install dependencies: flask, python-protobuf, protobuf_to_dict, stravalib (optional)
  * e.g., on Linux/Mac: ``pip install flask protobuf protobuf_to_dict stravalib``
  * e.g., on Windows in command prompt: ``C:\Python27\Scripts\pip.exe install flask protobuf protobuf_to_dict stravalib``
* Clone or download this repo
* Run standalone.py before starting Zwift
  * e.g., on Linux/Mac: ``sudo ./standalone.py``
    * sudo is needed because we're binding to the privileged ports 80 and 443.
  * e.g., on Windows in command prompt: ``C:\Python27\python.exe standalone.py``
* Start Zwift with standalone.py running (after completing step 2)

zoffline can be installed on the same machine as Zwift or another local machine.
</details>


<details><summary>Advanced usage (using Apache)</summary>
The easiest way to install zoffline with Apache is through
[Docker](https://www.docker.com/). zoffline can either be installed on the same
machine as Zwift or another local machine.

* Install Docker
* Create the docker container with:<br>
  ``docker create --name zwift-offline -p 443:443 -p 80:80 -v </path/to/host/storage>:/usr/local/apache2/htdocs/zwift-offline/storage -e TZ=<timezone> zoffline/zoffline``
  * You can optionally exclude ``-v </path/to/host/storage>:/usr/local/apache2/htdocs/zwift-offline/storage`` if you don't care if zoffline data is persistent across zoffline updates.
  * The path you pass to ``-v`` will likely need to be world readable and writable.
  * A list of valid ``<timezone>`` values (e.g. America/New_York) can be found [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).
  * Adding ``--restart unless-stopped`` will make zoffline start on boot if you have Docker v1.9.0 or greater.
* Start zoffline with:
  ``docker start zwift-offline``

If you don't use the Docker container you will need to set up an Apache server (or
write your own nginx/uWSGI configuration and use nginx) and install the
dependencies listed below. The necessary Apache configuration is inside the
``apache`` subdir.
</details>


### Step 2: Configure Zwift client to use zoffline

<details><summary>Windows 10 Instructions</summary>

* Install Zwift
  * If your Zwift version is newer than 1.0.31087 you may have to uninstall, then reinstall after installing zoffline.
  * If your Zwift version is 1.0.31087, you're all set.
  * If Zwift is not installed install it after installing zoffline (1.0.31087 will be installed instead of the latest).
* On your Windows machine running Zwift, copy the following files in this repo to a known location:
  * ``ssl/cert-zwift-com.p12``
  * ``ssl/cert-zwift-com.pem``
* Open Command Prompt as an admin, cd to that location and run
  * ``certutil.exe -importpfx Root cert-zwift-com.p12``
* Open Notepad as an admin and open ``C:\Program Files (x86)\Zwift\data\cacert.pem``
  * Append the contents of ``ssl/cert-zwift-com.pem`` to cacert.pem
* Open Notepad as an admin and open ``C:\Windows\System32\Drivers\etc\hosts``
  * Append this line: ``<zoffline ip> us-or-rly101.zwift.com secure.zwift.com cdn.zwift.com``
    <br />(Where ``<zoffline ip>`` is the ip address of the machine running zoffline. If
    it's running on the same machine as Zwift, use ``127.0.0.1`` as the ip.)

Why: We need to redirect Zwift to use zoffline and convince Windows and Zwift to
accept zoffline's self signed certificates for Zwift's domain names. Feel free
to generate your own certificates and do the same.

</details>

<details><summary>Mac OS X Instructions (Thanks @oldnapalm!)</summary>

* Install Zwift
  * If your Zwift version is newer than 1.0.31087 you may have to uninstall, then reinstall after installing zoffline.
  * If your Zwift version is 1.0.31087, you're all set.
  * If Zwift is not installed install it after installing zoffline (1.0.31087 will be installed instead of the latest).
* On your Mac machine running Zwift, copy the following files in this repo to a known location:
  * ``ssl/cert-zwift-com.p12``
  * ``ssl/cert-zwift-com.pem``
* Open Keychain Access, select "System" under "Keychains", select "Certificates" under "Category"
    * Click "File - Import Items..." and import ``ssl/cert-zwift-com.p12``
    * Right click "\*.zwift.com", select "Get Info" and under "Trust" choose "When using this certificate: Always Trust".
* Using a text editor open ``~/Library/Application Support/Zwift/data/cacert.pem``
  * Append the contents of ``ssl/cert-zwift-com.pem`` to cacert.pem
* Using a text editor (with admin privileges) open ``/etc/hosts``
  * Append this line: ``<zoffline ip> us-or-rly101.zwift.com secure.zwift.com cdn.zwift.com``
    <br />(Where ``<zoffline ip>`` is the ip address of the machine running zoffline. If
    it's running on the same machine as Zwift, use ``127.0.0.1`` as the ip.)

Why: We need to redirect Zwift to use zoffline and convince OS X and Zwift to
accept zoffline's self signed certificates for Zwift's domain names. Feel free
to generate your own certificates and do the same.

</details>

<details><summary>Android (requires a rooted device)</summary>

* Install Zwift on the device
* Append the contents of ``ssl/cert-zwift-com.pem`` to ``/data/data/com.zwift.zwiftgame/dataES/cacerts.pem`` on the device
  * Note: this file will only exist after the first run of Zwift since it's downloaded after the initial install
  * Simple approach to achieve this if your device doesn't have a text editor:
    * ``adb push ssl/cert-zwift-com.pem /data/data/com.zwift.zwiftgame/dataES/``
    * In ``adb shell``: ``cd /data/data/com.zwift.zwiftgame/dataES/``
    * In ``adb shell``: ``cat cert-zwift-com.pem >> cacerts.pem``
    * However you do it, ensure the permissions and ownership of the file remains the same.
* Modify the device's /etc/hosts file
  * Append this line: ``<zoffline ip> us-or-rly101.zwift.com secure.zwift.com cdn.zwift.com``
    <br />(Where ``<zoffline ip>`` is the ip address of the machine running zoffline. If
    it's running on the same machine as Zwift, use ``127.0.0.1`` as the ip.)
  * If no text editor on the device, recommend:
    * ``adb pull /etc/hosts``
    * (modify on PC)
    * ``adb push hosts /etc/hosts``
* Start Zwift and sign in using any email/password

Why: We need to redirect Zwift to use zoffline and convince Zwift to
accept zoffline's self signed certificates for Zwift's domain names. Feel free
to generate your own certificates and do the same.

</details>

#### Enabling/Disabling zoffline

To use Zwift online like normal, comment out or remove the line added to the ``hosts``
file before starting Zwift.


### Step 3 [OPTIONAL]: Obtain current Zwift profile

If you don't obtain your current Zwift profile before first starting Zwift with
zoffline enabled, you will be prompted to create a new profile (name, weight,
height, etc.)

To obtain your current profile:
* Run ``scripts/get_profile.py -u <your_zwift_username>``
* Move the resulting profile.bin (saved in whatever directory you ran get_profile.py in) into the ``storage`` directory.
  * If using Docker, move profile.bin into the path you passed to ``-v``


### Step 4 [OPTIONAL]: Obtain Strava API token

* Install dependencies: stravalib, six
  * e.g., on Linux/Mac: ``pip install stravalib six``
  * e.g., on Windows in command prompt: ``C:\Python27\Scripts\pip.exe install stravalib six``
* Get CLIENT_ID and CLIENT_SECRET from https://www.strava.com/settings/api
* Run ``scripts/strava_auth.py --client-id CLIENT_ID --client-secret CLIENT_SECRET``
* Open http://localhost:8000/ and authorize.
* Add the resulting access token to ``storage/strava_token.txt``


## Dependencies

Docker

-or-

* Python 2 (https://www.python.org/downloads/)
* Flask (http://flask.pocoo.org/)
  * ``pip install flask``
* python-protobuf
  * ``pip install protobuf``
* protobuf_to_dict (https://github.com/benhodgson/protobuf-to-dict)
  * ``pip install protobuf_to_dict``
* OPTIONAL: stravalib (https://github.com/hozn/stravalib)
  * ``pip install stravalib``
  * Add your Strava API token to ``storage/strava_token.txt``


## Known issues

* Segment results always show up as having just occurred.


## Note

Future Zwift updates may break zoffline until it's updated. While zoffline is
enabled Zwift updates will not be installed.

Don't expose zoffline to the internet, it was not designed with that in mind.


## Disclaimer

Zwift is a trademark of Zwift, Inc., which is not affiliated with the maker of
this project and does not endorse this project.

All product and company names are trademarks of their respective holders. Use of
them does not imply any affiliation with or endorsement by them.
