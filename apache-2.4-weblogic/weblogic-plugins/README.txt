===============================================================================
     Oracle WebLogic Server Proxy Plug-In 12c
     for Apache HTTP Server
===============================================================================

This plug-in allows Apache HTTP server to proxy the incoming HTTP
requests to the back-end WebLogic Managed Server(s)/Cluster(s).

This README contains basic instructions for quickly setting up this
plug-in. For detailed information on the plug-in please refer to
the online documentation for the plug-in available at
http://docs.oracle.com/middleware/12212/webtier/webtier-plugins.htm

Contents of the zip
===================
The plug-in zip distribution contains the following files:
 README.txt       (this file)
 bin/orapki       (orapki tool for configuring Oracle wallets)
 jlib/*.jar       (orapki helper java libraries)
 lib/mod_wl.so    (weblogic proxy module for Apache httpd 2.2)
 lib/mod_wl_24.so (weblogic proxy module for Apache httpd 2.4)
 lib/*.so         (helper libraries)

What's New?
===========
1. WebLogic Web Server Plug-In considers MD5 signed certificates as insecure,
   and hence are disabled by default. If you are using SSL to connect to
   WebLogic Server, and if the wallet contains any certificates signed with MD5,
   they should be replaced by SHA-2 signed certificates; otherwise, the server
   will fail to start. Refer to online documentation for help.

   Check if you have a MD5-signed certificate (must be to be done for all
   certificates present in the wallet):
    a) Use the following command to find the DN of the certificate ->
       ${PLUGIN_HOME}/bin/orapki wallet display -wallet <wallet__location>
    b) Export certificate present in wallet ->
       ${PLUGIN_HOME}/bin/orapki  wallet export -wallet <wallet_Location>
       -dn 'DN_string' -cert <certificate_file>
    c) Use keytool to check signature algorithm used to sign <certificate_file>
       exported in step b ->
       $JAVA_HOME/bin/keytool -printcert -file <certificate_file>
    
   Replace MD5 self-signed user certificates with SHA-2 self-signed user
   certificates:
    a) Remove MD5 user certificate present in wallet ->
       ${PLUGIN_HOME}/bin/orapki  wallet remove -wallet <wallet_location>
       -dn 'DN_string' -user_cert [-pwd <pwd>] | [-auto_login_only]
    b) If user certificate is self signed certificate, then the same needs to
       be removed from trusted and requested certificate lists as well (in the
       same order) ->
       ${PLUGIN_HOME}/bin/orapki wallet remove -wallet <wallet_location> -dn
       'DN_string' -trusted_cert [-pwd <pwd>] | [-auto_login_only]
       ${PLUGIN_HOME}/bin/orapki wallet remove -wallet <wallet_location> -dn
       'DN_string' -cert_req [-pwd <pwd>] | [-auto_login_only]
    c) Add a self signed user certificate signed with SHA-2 algorithm and a
       key size of 2048 to wallet ->
       ${PLUGIN_HOME}/bin/orapki wallet add -wallet <wallet_Location>
       -dn 'DN_String' -keysize 2048 -sign_alg sha256 -self_signed
       -validity 9125 [-pwd <pwd>] | [-auto_login_only]

   Replace MD5 CA-signed user certificate with SHA-2 CA-signed user certificate:
       If you are using a MD5 CA-signed user certificate, you may contact the CA
       to get a new CA-signed SHA-2 user certificate.

   Update MD5 certificates imported into the trusted certificates list:       
       If you have any trusted certificate, signed using MD5 signature algorithm,
       imported in your wallet, the certificate of the corresponding backend Weblogic
       server needs to be updated to use SHA-2 signature algorithm. Once updated,
       replace the MD5 trusted certificate in your wallet with the new updated
       certificate.

2. WebLogic Web Server Plug-In 12c supports Apache HTTP Server 2.4.x
   Web Server through a new plug-in module (mod_wl_24.so). So, you
   will need to load the WebLogic Web Server plug-in module 'mod_wl.so'
   with Apache HTTP Server 2.2.x and the WebLogic Web Server Plug-In 
   module 'mod_wl_24.so' with Apache HTTP Server 2.4.x. This is 
   typically done by editing the Apache HTTP Server configuration
   files(s).
   
3. WebLogic Server 12.1.2 support deploying WebSocket applications. 
   WebLogic Web Server Plug-In 12.2.1.x for Oracle HTTP Server 12.2.1.x
   and Apache HTTP Server 2.2.x and 2.4.x can now handle such 
   'WebSocket connection upgrade' requests and effectively proxy to
   WebSocket applications hosted within WebLogic Server 12.1.2 and later.
   Please refer to "Notes on WebSocket Proxy Configurations", below.
   
4. Following WebLogic Web Server Plug-In configuration parameters are now 
   introduced:

    a) 'WLMaxWebSocketClients' -> Limits the number of active WebSocket
                                  connections at any instant of time. 
                                  Default: Half of MaxClients (or MaxRequestWorkers).

    b) 'WebLogicSSLVersion'    -> Chooses the SSL Protocol version to use while 
                                  communicating HTTPS between Web Server
                                  and WebLogic Managed Server(s) / Cluster(s).
                                  (Applicable to Oracle HTTP Server and 
                                  Apache HTTP Server(s) only)
   
5. WebLogic Web Server Plug-In for Apache HTTP Server and Oracle iPlanet 
   Web Server can now log the debug information to the respective Web Server
   error log files. Hence the plug-in parameters specific to the debug 
   logs (Debug and WLLogFile) have been deprecated.

Notes on WebSocket Proxy Configurations
=======================================
1. If 'mod_reqtimeout' module is used within the Apache HTTP Server, 
   then the configured 'client time-out' value needs to be set
   appropriately to consider for the WebSocket connections.

2. The default timeout value for HTTP requests in 'mod_reqtimeout' 
   module has changed between Apache HTTP Server 2.2 and 2.4. This
   can cause the 'WebSocket' connections to break. So, you will
   need to use an appropriate 'client timeout' value.

3. You should configure appropriate 'client timeout' values for the
   WebSocket connections to avoid malicious attacks like a
   Denial Of Service attack.
   
Prerequisites
=============
1. JDK8  - JAVA_HOME
   (required for orapki tool usage only)

2. Apache HTTP Server 2.2.x - APACHE_HOME
   (apache listening on apache-host:apache-port)
                      OR
   Apache HTTP Server 2.4.x - APACHE_HOME
   (apache listening on apache-host:apache-port)

3. Plug-in zip extract location - PLUGIN_HOME
   (eg. /home/myhome/weblogic-plugins/)

4. WLS installation,  (could be on a different machine) - WL_HOME
   (listening on wls-host:wls-port,wls-secure-port)
   with some application deployed on the WLS instance.
   (eg. mywebapp with my.jsp). Please ensure you have
   WLS 12.1.2 or higher version if you are using WebSocket 
   connections.

Configuring the apache plug-in (for demo purposes only)
=======================================================
1. Make a copy of ${APACHE_HOME}/conf/httpd.conf file

2. Load the plug-in module

   A) With Apache HTTP Server 2.2.x: 
   ================================
   Edit the file and, add the following:

   ...
   LoadModule weblogic_module /home/myhome/weblogic-plugins/lib/mod_wl.so

   <IfModule mod_weblogic.c>
     WebLogicHost wls-host
     WebLogicPort wls-port
   </IfModule>

   <Location /mywebapp>
     SetHandler weblogic-handler
   </Location>
   ...

   B) With Apache HTTP Server 2.4.x: 
   ================================
   Edit the file and, add the following:
   ...
   LoadModule weblogic_module /home/myhome/weblogic-plugins/lib/mod_wl_24.so

   <IfModule mod_weblogic.c>
     WebLogicHost wls-host
     WebLogicPort wls-port
   </IfModule>

   <Location /mywebapp>
     SetHandler weblogic-handler
   </Location>
   ...

3. Ensure that the ${PLUGIN_HOME}/lib is included in the LD_LIBRARY_PATH:
   $ export LD_LIBRARY_PATH=/home/myhome/weblogic-plugins/lib:...

   (Other options include copying the 'lib' contents to APACHE_HOME/lib or
    editing the APACHE_HOME/bin/apachectl to update the LD_LIBRARY_PATH)

4. Start Apache HTTP Server:
   $ ${APACHE_HOME}/bin/apachectl start

5. Send a request to http://apache-host:apache-port/mywebapp/my.jsp
   from the browser. Validate the response.

Configuring SSL with WebLogic Server demo trust CA
===================================================
NOTE that this is for demo purposes only. When used in production, ensure that
trusted CAs are properly configured on the plug-in as well as on WebLogic
Server side.

The following steps will enable SSL between the plug-in and WLS:

1. Create an Oracle Wallet with orapki utility:
   (run this command on the system where the plug-in is being configured)
   $ ${PLUGIN_HOME}/bin/orapki wallet create -wallet my-wallet -auto_login_only

2. If the user who runs the Apache plug-in is not the same user who created the
   wallet (or has ROOT account access), the wallet creator would needs to grant
   access to the wallet by running the command chmod after creating the wallet.

   For example:

   $ chmod a+r <wallet_path>\cwallet.sso

3. Import the CA into the Oracle Wallet:
   Locate the Demo CA in WLS installation at ${WL_HOME}/sever/lib/CertGenCA.der
   $ ${PLUGIN_HOME}/bin/orapki wallet add -wallet my-wallet -trusted_cert
     -cert CertGenCA.der -auto_login_only

4. Enable SSL on the plug-in:
   Edit/Add the plug-in configuration in ${APACHE_HOME}/conf/httpd.conf as follows:

   ...
   WebLogicHost my-wls-ip
   WebLogicPort secure-port
   SecureProxy ON
   WLSSLWallet /home/myhome/mywallet
   ...

5. Send a request to http://apache-host:apache-port/mywebapp/my.jsp from the
   browser. Validate the response.

Two-way SSL
===========
There is no configuration on the plug-in to enable two-way SSL. The plug-in
will send a  user certificate (if it exists in the wallet) when WLS asks for
a user certificate. Use the following additional steps to setup two-way SSL:

1. Create a CSR (Certificate Signing Request) with the wallet.
2. Use the CSR to obtain a user certificate (self-signed or real CA signed)
3. Import the user certificate into the wallet.
4. Ensure that the CA (used to sign the user certificate) exists in the
   WLS trust store.
5. Enable WLS for two-way SSL.

Refer to online documentation for help on these steps.

Support and Patching
====================
When you encounter issues with the plug-in, ensure that you report the version
of the plug-in you are using to Oracle Support. You can find this information
in the apache log (if configured). The version information will appear like this:

WebLogic Server Plugin version 12.2.1.2.0, <WLSPLUGINS_XXXX_XXXX_XXXXX.XXXX>

You can also obtain the plug-in version, by issuing the following command

$ strings ${PLUGIN_HOME}/lib/mod_wl.so | grep -i wlsplugins

A patch for plug-in will typically contain one or more shared objects to be
replaced. Ensure you backup your original files as you replace them with the
files in the patch. Validate that the patch has been correctly updated by
checking the version string in the logs.

