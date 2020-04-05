# Free-Radius
Berikut cara install Free Radius di Linux Ubuntu

Install Server freeRadius

Recruietment
-	OS Ubuntu Server 12.04 LTS
-	FreeRADIUS Version 2.1.10

Installasi Paket
# apt get-install freeradius freeradius-ldap freeradius-utils

Konfigurasi Freeradius
Membuat virtual server
# cd /etc/freeradius/sites-available/
# vi virtual_11812
---------------------- start - virtual_11812 ------------------------
server  virtual_11812{
    	listen {
            	type = auth
            	ipaddr = *
            	port = 11812
            	$INCLUDE clients.conf
    	}

    	authorize {
            	preprocess
            	chap
            	mschap
            	digest
            	suffix
            	eap {
                    	ok = return
            	}
     #       	files
            	ldap_11812
            	expiration
            	logintime
            	pap
    	}
	authenticate {
            	Auth-Type PAP {
                    	pap
            	}
            	Auth-Type CHAP {
                    	chap
            	}
            	Auth-Type MS-CHAP {
                    	mschap
            	}
            	digest
            	unix
            	Auth-Type LDAP {
                    	ldap_11812
            	}
    	}

$INCLUDE    	${confdir}/sites-available/default
}
---------------------- end - virtual_11812 -------------------------

Mengaktifkan virtual server yang tadi sebelumnya kita buat
# cd ../sites-enabled/
# ln -s ../sites-available/virtual_11812 virtual_11812

Konfigurasi koneksi freeRadius ke server LDAP
# cd ../modules/
# vi ldap
-------------------------- start - ldap ----------------------------
ldap ldap_11812{
    	server = "10.0.9.21"
    	identity = "cn=admin,dc=tidore,dc=bm,dc=co,dc=id"
    	password = [PASSWORD]
    	basedn = "ou=hotspot_meeting,dc=tidore,dc=bm,dc=co,dc=id"
    	filter = "(uid=%u)"
    	ldap_connections_number = 5
    	timeout = 4
    	timelimit = 3
    	net_timeout = 1
    	tls {
            	start_tls = no
    	}
     	access_attr = "uid"
    	dictionary_mapping = ${confdir}/ldap.attrmap
    	edir_account_policy_check = no
}
--------------------------- end - ldap ------------------------------

# vi ../clients.conf
---------------------- start - clients.conf -----------------------
client 10.0.99.0/24 {
    	secret      	= testing123
    	shortname   	= localhost
    	nastype 		= other 	# localhost isn't usually a NAS...
}
------------------------ end - clients.conf -----------------------

Membuat service freeRadius tidak bisa di-kill
# cd /etc/init/
# vi freeradius.conf
--------------------- start - freeradius.conf ----------------------
# freeradius -- Radius server
#

description 	"Extensible, configurable radius daemon"
author      	"Michael Vogt <mvo@ubuntu.com>"

start on runlevel [2345]
stop on runlevel [!2345]

respawn

script
  if [ -r /etc/default/freeradius ]; then
	. /etc/default/freeradius
  fi
  /usr/sbin/freeradius -f $FREERADIUS_OPTIONS
end script

pre-start script
# /var/run may be a tmpfs
if [ ! -d /var/run/freeradius ]; then
  mkdir -p /var/run/freeradius
  chown freerad:freerad /var/run/freeradius
fi
end script
----------------------- end - freeradius.conf -----------------------

Refrensi
-	
