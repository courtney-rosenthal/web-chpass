# Installation Instructions for the "web-chpass" Package

You can view a formatted version of this document here:
https://github.com/chip-rosenthal/web-chpass/blob/master/INSTALL.md

## STEP 0: Prerequisites

To build and install this package, you will need the essential
build tools, as well the PAM development library. On Debian Linux
version 8 ("Jessie") I ran:

	$ sudo apt-get install build-essential libpam0g-dev

You also will need the Perl Template Toolkit. To install that
I ran:

	$ sudo apt-get install libtemplate-perl

## STEP 1: Configure the package

This is IMPORTANT.

There are several definitions in the Makefile to configure this package.
Edit the Makefile, read through the settings, and adjust them as needed.


## STEP 2: Build and install the package

Build by running:

	$ make

Install by running:

	$ sudo make install


## STEP 3: Test the nipasswd utility

The "nipasswd" utility performs the low-level password change operation.
It must be working before the web application can be used.

To test, run:

	$ sudo ./nipasswd -D -a
	USERNAME
	PASSWORD
	<CTRL/D>
	pam_conv_func>>> entered pam_conv_resp_count=0 num_msg=1
	pam_conv_func>>> msg_style="PAM_PROMPT_ECHO_OFF" msg="Password: " resp="foobar123"
	>>> Done!  Terminating with success exit status.

The undocumented -D option runs with debugging enabled. Once the program
is started, enter two lines: the username and password to test. Then type
CTRL/D.  The example shows that the user was authenticated successfully.

Here is an example to test changing a user password:

	$ sudo ./nipasswd -D
	USERNAME
	OLD_PASSWORD
	NEW_PASSWORD
	<CTRL/D>
	pam_conv_func>>> entered pam_conv_resp_count=0 num_msg=1
	pam_conv_func>>> msg_style="PAM_PROMPT_ECHO_OFF" msg="Password: " resp="foobar123"
	pam_conv_func>>> entered pam_conv_resp_count=1 num_msg=1
	pam_conv_func>>> msg_style="PAM_PROMPT_ECHO_OFF" msg="New password: " resp="foobar321"
	pam_conv_func>>> entered pam_conv_resp_count=2 num_msg=1
	pam_conv_func>>> msg_style="PAM_PROMPT_ECHO_OFF" msg="Retype new password: " resp="foobar321"
	>>> Done!  Terminating with success exit status.

Enter three lines: the username, the old password, and the new password
to set.  Then type CTRL/D.  Now you can verify that the password has
been changed.


## STEP 4: Install the web application

The "make install" step placed a "chpass" CGI script into your "cgi-bin"
directory, something like /usr/local/lib/cgi-bin. If your webserver
is not already configured to run CGI scripts from this directory, you
should set that up now.


If you are running Apache 2.4, for instance, configure this directory with:

	ScriptAlias /cgi-bin/ /usr/local/lib/cgi-bin/

	<Directory /usr/local/lib/cgi-bin/>
		AllowOverride None
		Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
		Require all granted
	</Directory>


## STEP 5: Test the application

Depending on how you configured the web application (step 4 above),
you can now run it by browsing:

	https://www.example.com/cgi-bin/chpass


## STEP 6: Customize the template

If you wish, you can customize the "chpass.tmpl" file.

The template is implemented with the Perl Template Toolkit
(http://template-toolkit.org/).

The template must produce a web page that submits (POST) a form with
the following parameters:

* action - Value must be "chpass".
* username
* old_passwd
* new_passwd
* new_passwd2

The available template variables are:

* message.content - If not empty, a status message to display.

* message.class - The class of the status message.  Either "alert"
  (default) or "success".

* form.enable - Boolean true if the change password form should
  be displayed.

* form.username - Default value of the username field.


## LDAP Notes

This package can be used even if the user databsae is in LDAP.

If your /etc/pam.d/passwd file is already setup for LDAP, that same
configuration will be used for "nipasswd".

You'll need to need to do is ensure that both "rootpwmoddn" and
"rootpwmodpw" are defined in your /etc/nslcd.conf file. That's needed
so the root user can change other users' passwords.

*SECURITY ISSUE* -- In a default Debian Linux environment, LDAP users are
allowed to set trivial passwrods. (That's because the "obscure" password
enforcement is part of the "pam_unix" module, which is not run for LDAP
password changing.) The fix is simple: install the "libpam-pwquality"
package.


## Author

This document is part of the "web-chpass" package.
https://github.com/chip-rosenthal/web-chpass

Chip Rosenthal
<chip@unicom.com>

