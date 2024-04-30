Topology
 - Control(ubuntu): NewRelic \ Wazuh \ Uptimerobot \ Pagerduty \ Splunk
 - Lux1(ubuntu): HTTP(nginx)
 - Lux2(RedHat): HTTPS(apache)
 - Win2022: IIS \ HTTP \ HTTPS
 - Win11: ?

1. Install Wazuh server on the control instance
    - curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a  (quick install on the wazuh's website for an updated command)
    - curl -so wazuh-passwords-tool.sh https://packages.wazuh.com/4.7/wazuh-passwords-tool.sh -> bash wazuh-passwords-tool.sh -u admin -p Secr3tP4ssw*rd <- reset the password (to change the password)
    - go to https://<control_ip>
    - add agent
    - choose the private IP to manage devices localy, or Public IP to manage remote devices(add their ips to the security group)
    - cp and paste the two commands at the end in the clients
    - after the agent is connected, click on him -> SCA -> click on the row at the bottom(the one with the fails and passes) -> harden the device
  
2. Installing Nginx into Lux1 | ref:https://ubuntu.com/tutorials/install-and-configure-nginx#2-installing-nginx
    - sudo apt update && sudo apt upgrade -y
    - sudo apt install nginx
    - go to http://<lux1_ip>

3. Installing Apache(httpd) on Lux2(redhat) | ref:https://access.redhat.com/documentation/en-us/jboss_enterprise_application_platform/6.3/html/administration_and_configuration_guide/install_the_apache_httpd_in_red_hat_enterprise_linux_with_jboss_eap_6_rpm
    - yum update
    - yum install httpd
    - systemctl enable httpd
    - systemctl start httpd
  
4. https apache on Lux2
    - yum install apache
    - yum install openssl
    - yum install mod_ssl
    - mkdir /etc/ssl/private
    - chmod 700 /etc/ssl/private
    - openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
    - systemctl restart httpd

5. New Relic Integration(key i got: NRAK-B5OJ5RED7FUMK3RMGFBAUFF9T53)
    - go to new relic, create an account, add the agent(copy the command into the client)
    - don't integrate with aws, the student account doesn't allow that
    - add new data source(apache, nginx, linux logs, etc.)
    - to add a new agent go to "add data" -> linux/windows OR apache/nginx, depends on what you want to be logged on that instance -> copy the command and paste it on the dsired client
  
6. Uptimerobot Integration
    - register -> new monitor -> add the webserver ip(use http instead of https when necessary) -> add the uptimerobot ips to the security group list with http(s) permissions so that they can probe the website(s)
  
7.  Pagerduty Integration with Wazuh(my acc: entarlc-1.pagerduty.com)
     ref: https://medium.com/@hasithaupekshitha97/streamlining-incident-response-wazuh-integration-with-pagerduty-989d7f5476da)
     ref: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/integration.html
    
    - register to pagerduty(use work/school acc and on US)
    - Log in to your PagerDuty account
    - Create a new service by navigating to Services > Add New Service.
    - Choose Events API v2 as the integration type
    - Copy the generated Integration Key
    - Access the Wazuh server and open the Wazuh configuration file located at /var/ossec/etc/ossec.conf.
    - Add the following configuration block within the <ossec_config> section:
          <integration>
            <name>pagerduty</name>
            <api_key>API_KEY</api_key> <!-- Replace with your PagerDuty API key -->
            <level>3<level> <!-- Level 3 for testing purpouses -->
            <alert_format>json</alert_format> <!-- With the new script this is mandatory -->
          </integration>

8. Configure IIS server on win2022
    - Install IIS, DNS Server, Certificate Authority
    - delete the default website and create one of your own(e.g www.bozo.com)
    - add that url to the dns server and change you dns server to 127.0.0.1
    - create a certificate request, store it, copy the contents, make a request to <ip>/certsrv
    - issue the certificate from the Certification Authority tab(tools -> Certification Authority)
    - go back to the certsvr website -> view pending requests, download the certificate(1st option)
    - install the certificate
    - go to iis -> bindings -> add https, choose the certificate and voila, all working localy, sort of

