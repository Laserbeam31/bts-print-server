BTS Print Server Documentation
==============================

The print server provides an "e-mail to print" function for the Office printer.

PDF Attachments sent to _printer@bts-crew.com_ are automatically downloaded by this server and are
sent to the Office printer.

E-mail account details:
-----------------------

The printer@bts-crew.com e-mail address is stored as an account on the Backstage Technical Services
GSuite. For the credentials of this account, please contact a committee member.

To allow automated access to the account by the print server over IMAP, an _App Password_ is configured
on this particular account. This is a separate password to the one used by regular (human) users, and
is _only_ used by the server for its automated connections.

Printer details:
----------------

The printer in the Office is a _Canon_ LBP6230dw.

This printer is registered on the University's campus _Docking_ wired network. This may be thought
of as "wired Eduroam". The printer's Docking IP address is: `138.38.112.82`.

Using Canon drivers, available for Windows, Mac, and Linux, documents can be sent directly to the
printer. However, this driver is tricky to make work reliably, and is a hassle to set up on
new PCs. The driver is also not available for mobile devices.

To mitigate the need for a driver on every BTS device which could conceivably be used to print
documents, the print server acts as an intermediary between devices and the printer. The server
"polls" the printer@bts-crew.com address regularly for any new e-mails with PDF attachments,
downloads the attachments, and sends them to the printer.

Server details:
---------------

The print server takes the form of a virtual machine (VM) on the SU server provided by Matt
Richards via DDaT. This virtual machine has a Docking IP address of `138.38.11.50`. The
VM runs Ubuntu Linux, with the Linux Canon drivers installed.

The VM may be accessed over SSH using the following credentials:

- Username: `su2bc`
- Password: `[REDACTED]`

All scripts/configurations relating to the e-mail-to-print functionality are stored in the
`/home/john/btsprintserver` directory.

Connection from the server to the printer is done using _CUPS_, the Common Unix Printing
System, as well as the aforementioned Canon drivers.

To poll the printer@bts-crew.com e-mail account, _Fetchmail_ is used. This is a basic Linux
IMAP e-mail client. The configuration file for fetchmail contains the settings used to poll
the relevant GSuite e-mail account and is located at `/home/john/fetchmail.conf`. It is in
this file where the _App Password_ (see above) is stored.

E-mail attachments are encoded in a peculiar way, and as such require special processing
to convert back into regular PDF files after having been downloaded in their "raw" form
by Fetchmail. _Uudeview_, a mail attachment extraction utility, is used for this.

The script used to actually poll the e-mail account, extract attachments, and send them to
the printer is a Bash script adapted from the following website:

https://blog.thomashampel.com/blog/tomcat2000.nsf/dx/print-email-attachments-with-a-raspberrypi.htm

This script is located at `/home/john/btsprintserver/printmail.sh` on the server.

When run, the script first polls the e-mail inbox for any new e-mails. If any new e-mails
are detected, they are downloaded and any PDF attachments are extracted using Uudeview.
Finally, using the `lpr` command, the attachments are sent to the printer.

Note that when executed, this script only runs a single time. A Crontab job on the VM
is therefore used to run the script repeatedly every few seconds, thus ensuring that
the inbox is regularly checked for new messages and that any PDFs get printed quickly.

