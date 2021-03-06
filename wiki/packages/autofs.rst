autofs
======

Mit diesem Paket können verschiedene Dateisysteme nach /var/media/autofs
gemountet werden.

Zweck ist, dass Mountpoints nur eingebunden werden, wenn darauf
zugegriffen wird. Nach einem Timeout ohne Zugriffe werden sie wieder
ausgehängt. Das funktioniert mit allen Dateisystemen und ist besonders
praktisch bei Netzwerk-Freigaben (NFS, CIFS, DAVFS etc.), da der Server,
auf den man zugreifen möchte, nicht bei Start des Freetz-Packages, das
diesen mountet, erreichbar sein muss, sondern nur bei Zugriff.

.. _OptionaleAufrufparameter:

Optionale Aufrufparameter
-------------------------

Zur Fehlersuche empfiehlt sich der Parameter ``-v`` und zusätzlich evtl
``-d``. Die Meldungen werden per `syslogd-cgi <syslogd.html>`__
ausgegeben.

.. _Beispielkonfigurationenderauto.conf:

Beispielkonfigurationen der auto.conf
-------------------------------------

.. _NFS:

NFS
~~~

Für NFS wird lediglich das Modul nfs.ko benötigt.

.. code:: bash

   NFS-SHARE -rw,soft,intr,rsize=8192,wsize=8192         SERVER:/SHARE

.. _Samba:

Samba
~~~~~

Hierfür wird das Paket `cifsmount <cifsmount.html>`__ benötigt, dessen
Webinterface nicht.

.. code:: bash

   SMB-SHARE -fstype=cifs,user=USER,pass=PASS,ro         ://SERVER/SHARE

.. _WebDAV:

WebDAV
~~~~~~

Für WebDAV wird das `davfs2 <davfs2.html>`__-Paket (ohne Webinterface)
benötigt.

.. code:: bash

   DAV-SHARE -fstype=davfs,conf=/tmp/flash/autofs/davfs.conf     :http\://SERVER

Außerdem noch diese 2 Dateien je Mountpoint:

/tmp/flash/autofs/davfs.conf

.. code:: bash

   secrets /tmp/flash/autofs/davfs.secrets
   ask_auth 0
   #falls benötigt:
   #if_match_bug 1

/tmp/flash/autofs/davfs.secrets (Dateirechte 600!)

.. code:: bash

   http://SERVER USERNAME    PASSWORT

.. _SSHfs:

SSHfs
~~~~~

Es werden die Packages OpenSSH und SSHfs-FUSE benötigt

.. code:: bash

   SSH-SHARE -fstype=fuse,rw,allow_other             :sshfs\#USER@SERVER\:/

Außerdem muss der Server in der known_hosts bekannt sein und in id_rsa
oder id_dsa muss der private Schlüssel hinterlegt sein. Diese Dateien
können mit dem SSH/\ `authorized_keys <authorized-keys.html>`__ Package
bearbeitet werden

.. _HTTPsFTPsLDAP:

HTTP(s), FTP(s), LDAP
~~~~~~~~~~~~~~~~~~~~~

Momentan nicht möglich, da
`​CurlFtpFS <http://sourceforge.net/projects/curlftpfs/>`__ momentan
nicht in Freetz verfügbar ist. Siehe auch `​archLinux
Wiki <https://wiki.archlinux.org/index.php/Autofs>`__.
