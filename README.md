# Odoo_Deploy_scripts
------------------------------------------------------------------------------------------------------

Method-1 (Install Odoo in Ubuntu - Using Odoo_Install.sh)
=============================================================
Source: github.com/Yenthe666/InstallScript/tree/17.0

`Input:`

	sudo wget https://raw.githubusercontent.com/Yenthe666/InstallScript/17.0/odoo_install.sh
	sudo nano odoo_install.sh
	sudo chmod +x odoo_install.sh
	sudo ./odoo_install.sh

`Output:`

	Done! The Odoo server is up and running. Specifications:
	Port: 8069
	User service: odoo
	Configuraton file location: /etc/odoo-server.conf
	Logfile location: /var/log/odoo
	User PostgreSQL: odoo
	Code location: odoo
	Addons folder: odoo/odoo-server/addons/
	Password superadmin (database): show_pass_here
	Start Odoo service: sudo service odoo-server start
	Stop Odoo service: sudo service odoo-server stop
	Restart Odoo service: sudo service odoo-server restart



Method-2 (Setup Odoo on Ubuntu using commands):
=============================================================================
`Source:`

	https://zacs-tech.com/how-to-install-odoo-17-on-ubuntu-22-04/
	https://www.youtube.com/watch?v=jDPog61i0Hw&ab_channel=ZacsTech

sudo apt update

sudo apt upgrade -y

sudo apt install build-essential wget git python3.11-dev python3.11-venv libfreetype-dev libxml2-dev libzip-dev libsasl2-dev node-less libjpeg-dev zlib1g-dev libpq-dev libxslt1-dev libldap2-dev libtiff5-dev libopenjp2-7-dev libcap-dev -y

sudo /usr/sbin/adduser --system --shell /bin/bash --gecos "Odoo user" --group --home /opt/odoo17 odoo17

sudo apt install postgresql -y

sudo su - postgres -c "createuser -s odoo17"

sudo apt install wkhtmltopdf -y

sudo su - odoo17

git clone https://www.github.com/odoo/odoo --depth 1 --branch 17.0 odoo17

python3.11 -m venv odoo17-venv

source odoo17-venv/bin/activate

pip3 install wheel setuptools pip --upgrade

pip3 install -r odoo17/requirements.txt

mkdir /opt/odoo17/odoo17/custom-addons

exit

sudo nano /etc/odoo17.conf
===================================
	[options]
	admin_passwd = m0d1fyth15
	db_host = False
	db_port = False
	db_user = odoo17
	db_password = False
	addons_path = /opt/odoo17/odoo17/addons,/opt/odoo17/odoo17/custom-addons

sudo nano /etc/systemd/system/odoo17.service
==============================================
	[Unit]
	Description=odoo17
	Requires=postgresql.service
	After=network.target postgresql.service
	[Service]
	Type=simple
	SyslogIdentifier=odoo17
	PermissionsStartOnly=true
	User=odoo17
	Group=odoo17
	ExecStart=/opt/odoo17/odoo17-venv/bin/python3 /opt/odoo17/odoo17/odoo-bin -c /etc/odoo17.conf
	StandardOutput=journal+console
	[Install]
	WantedBy=multi-user.target



sudo systemctl daemon-reload

sudo systemctl start odoo17

sudo systemctl status odoo17

sudo journalctl -u odoo17

sudo systemctl enable odoo17

sudo systemctl enable --now odoo17

sudo systemctl status odoo17

sudo systemctl status postgresql

sudo systemctl start postgresql

sudo journalctl -u postgresql

sudo systemctl restart odoo17



sudo apt install nginx -y

sudo nano /etc/nginx/conf.d/odoo.conf
======================================
	upstream odoo17 {
	 server 127.0.0.1:8069;
	}
	
	upstream odoochat {
	 server 127.0.0.1:8072;
	}
	
	server {
	    listen 80;
	    server_name domain.com;  
	
	    
	    # log files
	    access_log /var/log/nginx/odoo17.access.log;
	    error_log /var/log/nginx/odoo17.error.log;
		
	    proxy_buffers 16 64k;
	    proxy_buffer_size 128k;
	
	location / {
	    proxy_pass http://odoo17;
	    proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
	    proxy_redirect off;
	
	    # Proxy headers
	    proxy_set_header Host $host;
	    proxy_set_header X-Real-IP $remote_addr;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header X-Forwarded-Proto $scheme;
	}
	
	# Handle longpoll requests
	location /longpolling {
	        proxy_pass http://odoochat;
	    }
	
	
	# Cache static files
	location ~* /web/static/ {
	        proxy_cache_valid 200 60m;
	        proxy_buffering on;
	        expires 864000;
	        proxy_pass http://odoo17;
	    }
	
	}


sudo systemctl restart nginx

sudo apt install certbot python3-certbot-nginx -y

sudo certbot --nginx -d domain.com






Method-3(Automatic aws odoo deployment with django code):
==============================================================================================
	from django.shortcuts import render, redirect, HttpResponse
	from User_Registration_App.models import CompanyRegistrationInformation
	from django.shortcuts import render, redirect
	from django.contrib.auth.decorators import login_required
	from .models import OdooDomainSetup
	from subscription_app.models import SubscriptionInformation
	import boto3
	import os
	import psycopg2
	from django.contrib import messages
	import time
	import uuid
	import paramiko
	from django.shortcuts import get_object_or_404, redirect
	from psycopg2 import sql  # Import the sql module from psycopg2
	from io import StringIO
	
	
	
	
	@login_required
	def addMyDomain(request, pk):
	    if request.user.is_authenticated:
	        company_info = CompanyRegistrationInformation.objects.get(id=pk)
	
	        Added_Domains = OdooDomainSetup.objects.filter(user=request.user)
	        context={'company_info':company_info, "user_info":request.user, 'Added_Domains':Added_Domains}
	        return render(request, "Automatic_Deployment/addMyDomain.html", context)
	    else:
	        return redirect('login_info')
	
	
	
	@login_required
	def get_static_ip_view(request):
	    if request.method == 'POST':
	        domain = request.POST.get('domain')
	        company_info_id = request.POST.get('company_info_id')
	
	        domain_name = domain.replace(".", "_")
	
	        subcription = SubscriptionInformation.objects.filter(company_info=company_info_id).last()
	        erp_users = int(subcription.number_of_expected_users_of_the_platform)
	
	        try:
	            print('Verifying aws client ...')
	            # Allocate static IP
	            client = boto3.client(
	                'lightsail',
	                region_name=os.getenv('AWS_REGION'),
	                aws_access_key_id=os.getenv('S3_ACCESS_KEY_ID'),
	                aws_secret_access_key=os.getenv('S3_SECRET_ACCESS_KEY')
	            )
	            print('Successfully got the aws client!')
	        except Exception as e:
	            print(f"Failed to get the aws client according aws credentials : {e}")
	            messages.error(request, f"Failed to get the aws client according aws credentials : {e}")
	            return redirect('addMyDomain', company_info_id)
	
	        try:
	            print('Trying Allocate static IP ...')
	            # Allocate static IP
	            staticIpName = "Odoo-" + str(domain_name) + str("-StaticIp")
	            static_ip_response = client.allocate_static_ip(staticIpName=staticIpName)
	            static_ip_name = static_ip_response['operations'][0]['resourceName']
	            print('Successfully Allocate static IP !')
	        except Exception as e:
	            print(f'Failed to Allocate static IP : {e}')
	            messages.error(request, f'Failed to Allocate static IP : {e}')
	            return redirect('addMyDomain', company_info_id)
	
	        try:
	            print('Trying Retrieve the allocated IP to get the address ...')
	            # Retrieve the allocated IP to get the address
	            static_ip_details = client.get_static_ip(staticIpName=static_ip_name)
	            static_ip_address = static_ip_details['staticIp']['ipAddress']
	            print(f'Successfully Retrieve the allocated IP{static_ip_address} to get the address !')
	        except Exception as e:
	            static_ip_address = '0.0.0.0'
	            print(f'Failed to Retrieve the allocated IP to get the address : {e}')
	
	
	        setup = OdooDomainSetup(
	            user=request.user,
	            domain=domain,
	            erp_users=erp_users,
	            static_ip_name=static_ip_name,
	            static_ip=static_ip_address,
	            blueprint_id='ubuntu_22_04',  # Replace with your desired blueprint ID
	        )
	        setup.save()
	        setup_id = setup.id
	        return redirect('confirm_ip', company_info_id, setup_id)
	
	
	
	@login_required
	def confirm_ip_view(request, company_info_id, setup_id):
	    setup = OdooDomainSetup.objects.get(id=setup_id)
	    if request.method == 'POST':
	        setup.confirmed = True
	        setup.save()
	        print('Confirmed ip and pointed to domain ...')
	        setup_id = setup.id
	        return redirect('create_instance', company_info_id, setup_id)
	
	    company_info = CompanyRegistrationInformation.objects.get(id=company_info_id)
	    print('Waiting for confirming ip and point to domain ...')
	    return render(request, 'Automatic_Deployment/confirm_ip.html', {'setup': setup, 'company_info': company_info, "user_info": request.user})
	
	
	def verify_instance_key_pair(instance_name, expected_key_pair_name):
	    try:
	        client = boto3.client(
	            'lightsail',
	            region_name=os.getenv('AWS_REGION'),
	            aws_access_key_id=os.getenv('S3_ACCESS_KEY_ID'),
	            aws_secret_access_key=os.getenv('S3_SECRET_ACCESS_KEY')
	        )
	        response = client.get_instance(instanceName=instance_name)
	        instance_key_pair_name = response['instance']['sshKeyName']
	
	        if instance_key_pair_name == expected_key_pair_name:
	            print(f"Instance '{instance_name}' is associated with key pair '{expected_key_pair_name}'.")
	            return True
	        else:
	            print(f"Instance '{instance_name}' is associated with key pair '{instance_key_pair_name}', not '{expected_key_pair_name}'.")
	            return False
	    except Exception as e:
	        print(f"Error verifying instance key pair: {e}")
	        return False
	
	
	@login_required
	def create_instance_view(request, company_info_id, setup_id):
	    print('Processing for create instance ...')
	    setup = OdooDomainSetup.objects.get(id=setup_id)
	    if not setup.confirmed:
	        return redirect('confirm_ip', setup_id=setup.id)
	
	    domain_name = setup.domain.replace(".", "_")
	    unique_suffix = str(uuid.uuid4()).split('-')[0]  # Shorten the UUID for length constraints
	
	    # Determine Lightsail instance size based on erp_users
	    if setup.erp_users <= 10:
	        bundle_id = 'small_1_0'
	    elif setup.erp_users <= 25:
	        bundle_id = 'medium_1_0'
	    else:
	        bundle_id = 'large_1_0'
	
	    # Connect to Lightsail
	    client = boto3.client(
	        'lightsail',
	        region_name=os.getenv('AWS_REGION'),
	        aws_access_key_id=os.getenv('S3_ACCESS_KEY_ID'),
	        aws_secret_access_key=os.getenv('S3_SECRET_ACCESS_KEY')
	    )
	    if setup.key_pair_name:
	        print('Already Have KeyPair for this domain')
	    else:
	        try:
	            key_pair_name = f"Odoo-{domain_name[:20]}-{unique_suffix}-keypair"
	            print('Trying to create new key pair ...')
	            # Create key pair
	            key_pair_response = client.create_key_pair(
	                keyPairName=key_pair_name
	            )
	            private_key = key_pair_response['privateKeyBase64']
	
	            # Save the private key in the database
	            setup.private_key = private_key
	            setup.key_pair_name = key_pair_name
	            setup.save()
	
	            print(f"Key pair '{key_pair_name}' created and saved name and private key to the database.")
	
	        except Exception as e:
	            print(f"Failed to create key pair: {e}")
	            messages.error(request, "Failed to create key pair.")
	            return redirect('addMyDomain', company_info_id)
	    if setup.instance_name:
	        print("Instance Already Created for this domain")
	    else:
	        try:
	            print('Trying to create new instance ...')
	            instance_name = f"Odoo-{domain_name}-instanceName"
	            # Create instance
	            instance_response = client.create_instances(
	                instanceNames=[instance_name],
	                availabilityZone=f'{os.getenv("AWS_REGION")}a',
	                blueprintId='ubuntu_22_04',  # Replace with your desired blueprint ID
	                bundleId=bundle_id,
	                keyPairName=key_pair_name
	            )
	            print(f'Successfully created new instance {instance_name}...')
	
	            # Extract instance ID
	            instance_id = instance_response['operations'][0]['resourceName']
	            setup.instance_name = instance_name
	            setup.instance_id = instance_id
	            setup.bundle_id = bundle_id
	            setup.save()
	
	        except Exception as e:
	            print(f'Failed to create instance: {e}')
	            messages.error(request, "Failed to create instance !")
	            return redirect('addMyDomain', company_info_id)
	
	
	    # Wait for the instance to be in "running" state
	    instance_state = ""
	    while instance_state != "running":
	        try:
	            instance_details = client.get_instance(instanceName=setup.instance_name)
	            instance_state = instance_details['instance']['state']['name']
	            print(f"Instance state: {instance_state}")
	            if instance_state == "running":
	                break
	            else:
	                time.sleep(10)  # Wait for 10 seconds before checking again
	        except Exception as e:
	            print(f"Error retrieving instance state: {e}")
	            time.sleep(10)  # Retry after 10 seconds if there's an error
	
	    try:
	        print(f"Trying to Attach static IP with Instance ({setup.static_ip_name} + {setup.instance_name})...")
	        # Attach static IP
	        client.attach_static_ip(
	            staticIpName=setup.static_ip_name,
	            instanceName=setup.instance_name
	        )
	        print(f"Successfully Attached static IP with Instance ! ({setup.static_ip_name} + {setup.instance_name})")
	    except Exception as e:
	        print(f"Failed to Attach static IP with Instance: {e}")
	
	    # Get the public IP address of the instance
	    try:
	        instance_details = client.get_instance(instanceName=instance_name)
	        instance_public_ip = instance_details['instance']['publicIpAddress']
	        print(f"Instance Public IP : {instance_public_ip}")
	        setup.instance_public_ip = instance_public_ip
	        setup.save()
	    except Exception as e:
	        print(f"Failed to retrieve instance public IP: {e}")
	
	    # Add firewall rules for ports 22, 80-81, 443, and 8069
	    try:
	        print("Trying to add firewall rules for ports 22, 80-81, 443, and 8069 ...")
	        client.put_instance_public_ports(
	            portInfos=[
	                {
	                    'fromPort': 22,
	                    'toPort': 22,
	                    'protocol': 'TCP'
	                },
	                {
	                    'fromPort': 80,
	                    'toPort': 81,
	                    'protocol': 'TCP'
	                },
	                {
	                    'fromPort': 443,
	                    'toPort': 443,
	                    'protocol': 'TCP'
	                },
	                {
	                    'fromPort': 8069,
	                    'toPort': 8069,
	                    'protocol': 'TCP'
	                },
	            ],
	            instanceName=instance_name
	        )
	        print("Successfully added firewall rules for ports 22, 80-81, 443, and 8069.")
	    except Exception as e:
	        print(f"Failed to add firewall rules: {e}")
	
	    company_info = CompanyRegistrationInformation.objects.get(id=company_info_id)
	    return render(request, 'Automatic_Deployment/instance_created.html', {'setup': setup, "company_info": company_info, "user_info": request.user})
	
	
	
	@login_required
	def setup_odoo_docker_view(request, setup_id, company_info_id):
	    try:
	        setup = OdooDomainSetup.objects.get(id=setup_id)
	
	        unique_suffix = str(uuid.uuid4()).split('-')[0]
	        # db_name = f"{setup.domain.replace('.', '_')}_{unique_suffix}_db"
	        db_user = f"{setup.domain.replace('.', '_')}_{unique_suffix}"
	        db_password = f"{setup.domain.replace('.', '_')}_{unique_suffix}"
	        # setup.db_name = db_name
	        setup.db_user = db_user
	        setup.db_password = db_password
	        setup.save()
	
	        instance_public_ip = setup.instance_public_ip
	
	        # Debug statements to check setup values
	        print(f"Instance Public IP: {instance_public_ip}")
	        print(f"DB User: {setup.db_user}")
	        # print(f"DB Password: {setup.db_password}")
	
	        # Create PostgreSQL user
	        if not create_postgresql_user(setup.db_user, setup.db_password):
	            messages.error(request, "Failed to create PostgreSQL user.")
	            return redirect('addMyDomain', company_info_id)
	
	        # Clone Odoo repository
	        clone_odoo_repo_commands = [
	            'git clone https://github.com/odoo/odoo.git -b 17.0 --depth 1 /home/ubuntu/odoo'
	        ]
	        if not ssh_execute_command(instance_public_ip, 'ubuntu', setup.private_key, clone_odoo_repo_commands):
	            messages.error(request, "Failed to clone Odoo repository.")
	            return redirect('addMyDomain', company_info_id)
	
	        # Docker Compose and Nginx configuration content
	        docker_compose_content = f"""
	version: '3.9'
	services:
	    odoo:
	        image: odoo:17.0
	        restart: always
	        tty: true
	        command: -c /etc/odoo/odoo.conf
	        volumes:
	            - ./custom_addons:/mnt/extra-addons
	            - ./config:/etc/odoo
	            - /home/ubuntu/odoo:/mnt/odoo
	            - odoo_data:/var/lib/odoo
	        ports:
	            - "8069:8069"
	            - "8072:8072"  # Expose Odoo longpolling port
	        networks:
	            - odoo-network
	networks:
	  odoo-network:
	volumes:
	  odoo_data:
	"""
	
	
	
	#         docker_compose_content = f"""
	# version: '3.9'
	# services:
	#     odoo:
	#         image: odoo:17.0
	#         restart: always
	#         tty: true
	#         command: -c /etc/odoo/odoo.conf
	#         volumes:
	#             - ./custom_addons:/mnt/extra-addons
	#             - ./config:/etc/odoo
	#             - /home/ubuntu/odoo:/mnt/odoo
	#             - odoo_data:/var/lib/odoo
	#         ports:
	#             - "8069:8069"
	#             - "8072:8072"  # Expose Odoo longpolling port
	#         networks:
	#             - odoo-network
	#     db:
	#         image: postgres:latest
	#         environment:
	#             POSTGRES_DB: {db_name}
	#             POSTGRES_USER: {db_user}
	#             POSTGRES_PASSWORD: {db_password}
	#         volumes:
	#             - db_data:/var/lib/postgresql/data
	#         networks:
	#             - odoo-network
	# networks:
	#   odoo-network:
	# volumes:
	#   odoo_data:
	#   db_data:
	# """
	
	
	        # Docker Compose and Nginx configuration content
	#         docker_compose_content = f"""
	# version: '3.9'
	# services:
	#     odoo:
	#         image: odoo:17.0
	#         restart: always
	#         tty: true
	#         volumes:
	#              - ./custom_addons:/mnt/extra-addons
	#              - ./config:/etc/odoo
	#              - odoo_data:/var/lib/odoo
	#         ports:
	#              - "8069:8069"
	#              - "8072:8072"
	# volumes:
	#     odoo_data:
	# """
	
	
	
	
	        odoo_conf_content = f"""
	[options]
	admin_passwd = kintah
	db_host = {os.getenv('MS_DB_HOST')}
	db_user = {setup.db_user}
	db_password = {setup.db_password}
	addons_path = /mnt/extra-addons,/mnt/odoo/addons
	
	data_dir = /var/lib/odoo
	proxy_mode = True
	dbfilter = .*
	http_port = 8069
	longpolling_port = 8072
	limit_memory_hard = 1677721600
	limit_memory_soft = 629145600
	limit_request = 8192
	limit_time_cpu = 600
	limit_time_real = 1200
	max_cron_threads = 1
	workers = 5
	
	logfile = /var/log/odoo/odoo-server.log
	"""
	
	        # odoo_conf_content = f"""
	        # [options]
	        # ; This is the password that allows database operations:
	        # admin_passwd = admin
	        # addons_path = /mnt/extra-addons
	        # data_dir = /var/lib/odoo
	        # db_host = {os.getenv('MS_DB_HOST')}
	        # db_port = {os.getenv('MS_DB_PORT')}
	        # db_user = {setup.db_user}
	        # db_password = {setup.db_password}
	        # proxy_mode = True
	        # ; dbfilter = .*
	        # xmlrpc_port = 8069
	        # longpolling_port = 8072
	        # ; limit_memory_hard = 1677721600
	        # ; limit_memory_soft = 629145600
	        # limit_request = 8192
	        # limit_time_cpu = 600
	        # limit_time_real = 1200
	        # max_cron_threads = 1
	        # workers = 5
	        # """
	
	
	
	
	
	        nginx_conf_content = f"""
	upstream odoo {{
	 server 127.0.0.1:8069;
	}}
	
	upstream odoochat {{
	 server 127.0.0.1:8072;
	}}
	
	server {{
	    listen 80;
	    server_name {setup.domain} www.{setup.domain};  # Replace with actual domain
	
	    proxy_read_timeout 720s;
	    proxy_connect_timeout 720s;
	    proxy_send_timeout 720s;
	
	    # Proxy headers
	    proxy_set_header X-Forwarded-Host $host;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header X-Forwarded-Proto $scheme;
	    proxy_set_header X-Real-IP $remote_addr;
	
	    # log files
	    access_log /var/log/nginx/odoo.access.log;
	    error_log /var/log/nginx/odoo.error.log;
	
	    # Handle longpoll requests
	    location /longpolling {{
	        proxy_pass http://odoochat;
	    }}
	
	    # Handle / requests
	    location / {{
	       proxy_redirect off;
	       proxy_pass http://odoo;
	    }}
	
	    # Cache static files
	    location ~* /web/static/ {{
	        proxy_cache_valid 200 90m;
	        proxy_buffering on;
	        expires 864000;
	        proxy_pass http://odoo;
	    }}
	
	    # Gzip
	    gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
	    gzip on;
	}}
	"""
	
	
	
	
	        # Commands to be executed on the remote server
	        commands = [
	            'sudo apt update',
	            'sudo apt install -y docker.io docker-compose nginx python3-certbot-nginx',
	            'sudo systemctl start docker',
	            'sudo systemctl enable docker',
	            'sudo docker rm -f db odoo || true',
	            'sudo docker network create odoo-network || true',
	            'mkdir -p odoo1/config odoo1/custom_addons',
	        ]
	
	        if ssh_execute_command(instance_public_ip, 'ubuntu', setup.private_key, commands):
	            # Create and transfer Docker Compose file
	            if not ssh_transfer_file(instance_public_ip, 'ubuntu', setup.private_key, 'docker-compose.yml',
	                                     docker_compose_content, 'odoo1/docker-compose.yml'):
	                messages.error(request, "Failed to transfer Docker Compose file.")
	                return redirect('addMyDomain', company_info_id)
	
	            # Create and transfer Odoo config file
	            if not ssh_transfer_file(instance_public_ip, 'ubuntu', setup.private_key, 'odoo.conf', odoo_conf_content,
	                                     'odoo1/config/odoo.conf'):
	                messages.error(request, "Failed to transfer Odoo config file.")
	                return redirect('addMyDomain', company_info_id)
	
	            # Run Docker Compose
	            run_docker_cmd = 'cd odoo1 && sudo docker-compose up -d'
	            if not ssh_execute_command(instance_public_ip, 'ubuntu', setup.private_key, [run_docker_cmd]):
	                messages.error(request, "Failed to start Docker containers.")
	                return redirect('addMyDomain', company_info_id)
	
	            # Remove default Nginx config
	            remove_nginx_default_cmd = 'sudo rm /etc/nginx/sites-enabled/default || true'
	            if not ssh_execute_command(instance_public_ip, 'ubuntu', setup.private_key, [remove_nginx_default_cmd]):
	                messages.error(request, "Failed to remove default Nginx config.")
	                return redirect('addMyDomain', company_info_id)
	
	            # Create and transfer Nginx config file
	            if not ssh_transfer_file(instance_public_ip, 'ubuntu', setup.private_key, 'nginx.conf', nginx_conf_content,
	                                     '/etc/nginx/sites-available/odoo.conf'):
	                messages.error(request, "Failed to transfer Nginx config file.")
	                return redirect('addMyDomain', company_info_id)
	
	            # Create symlink for Nginx config
	            nginx_commands = [
	                'sudo ln -s /etc/nginx/sites-available/odoo.conf /etc/nginx/sites-enabled/',
	                'sudo nginx -t',
	                'sudo systemctl restart nginx.service',
	                f'sudo certbot --nginx -d {setup.domain} -d www.{setup.domain} --register-unsafely-without-email --agree-tos --no-eff-email'
	            ]
	            if not ssh_execute_command(instance_public_ip, 'ubuntu', setup.private_key, nginx_commands):
	                messages.error(request, "Failed to configure Nginx.")
	                return redirect('addMyDomain', company_info_id)
	
	            # Check if Docker containers are running
	            check_docker_cmd = 'sudo docker ps --format "{{.Names}}"'
	            container_names = ssh_execute_command(instance_public_ip, 'ubuntu', setup.private_key, [check_docker_cmd],
	                                                  get_output=True)
	            # if 'odoo' in container_names and 'db' in container_names:
	            if 'odoo' in container_names:
	                print('Odoo deployment successful.')
	                setup.deployed = True
	                setup.save()
	                messages.success(request, f"Odoo deployment successful.")
	            else:
	                print('Docker containers not running as expected.')
	                messages.error(request, "Docker containers not running as expected.")
	                return redirect('addMyDomain', company_info_id)
	        else:
	            print('Odoo deployment failed.')
	            messages.error(request, "Odoo deployment failed.")
	            return redirect('addMyDomain', company_info_id)
	
	    except OdooDomainSetup.DoesNotExist:
	        messages.error(request, "Setup not found.")
	        return redirect('addMyDomain', company_info_id)
	    except Exception as e:
	        print(f"Error during deployment: {e}")
	        messages.error(request, "Error during deployment.")
	        return redirect('addMyDomain', company_info_id)
	
	    company_info = CompanyRegistrationInformation.objects.get(id=company_info_id)
	    return render(request, 'Automatic_Deployment/successpage.html',
	                  {'setup': setup, 'company_info': company_info, 'user_info': request.user})
	
	
	def ssh_transfer_file(instance_ip, username, private_key_string, local_file_name, file_content, remote_file_path):
	    try:
	        private_key_file = StringIO(private_key_string)
	        ssh_key = paramiko.RSAKey.from_private_key(private_key_file)
	
	        ssh_client = paramiko.SSHClient()
	        ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
	        ssh_client.connect(instance_ip, username=username, pkey=ssh_key)
	        print("SSH Connected")
	
	        sftp_client = ssh_client.open_sftp()
	        temp_remote_file_path = f"/tmp/{local_file_name}"
	        with sftp_client.file(temp_remote_file_path, 'w') as remote_file:
	            remote_file.write(file_content.encode())
	            remote_file.chmod(0o644)
	
	        # Move the file to the desired location using sudo
	        move_command = f"sudo mv {temp_remote_file_path} {remote_file_path}"
	        ssh_execute_command(instance_ip, username, private_key_string, [move_command])
	
	        sftp_client.close()
	        ssh_client.close()
	        print(f"File {local_file_name} transferred successfully to {remote_file_path}.")
	        return True
	    except Exception as e:
	        print(f"File transfer failed: {e}")
	        return False
	
	
	def create_postgresql_user(db_user, db_password):
	    try:
	        connection = psycopg2.connect(
	            host=os.getenv('MS_DB_HOST'),
	            port=os.getenv('MS_DB_PORT'),
	            user=os.getenv('MS_DB_USER'),
	            password=os.getenv('MS_DB_PASSWORD')
	        )
	        connection.autocommit = True
	        cursor = connection.cursor()
	
	        cursor.execute(sql.SQL("SELECT 1 FROM pg_roles WHERE rolname=%s"), [db_user])
	        user_exists = cursor.fetchone()
	
	        if not user_exists:
	            cursor.execute(sql.SQL("CREATE USER {} WITH PASSWORD %s;").format(
	                sql.Identifier(db_user)
	            ), [db_password])
	
	        cursor.execute(sql.SQL("ALTER USER {} CREATEDB;").format(
	            sql.Identifier(db_user)
	        ))
	
	        cursor.close()
	        connection.close()
	        print(f"User {db_user} created or verified successfully.")
	        return True
	
	    except Exception as e:
	        print(f"Error creating or verifying user: {e}")
	        return False
	
	
	def ssh_execute_command(instance_ip, username, private_key_string, commands, get_output=False):
	    try:
	        private_key_file = StringIO(private_key_string)
	        ssh_key = paramiko.RSAKey.from_private_key(private_key_file)
	
	        ssh_client = paramiko.SSHClient()
	        ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
	        ssh_client.connect(instance_ip, username=username, pkey=ssh_key)
	        print("SSH Connected")
	
	        output = ""
	        for command in commands:
	            print(f"Executing command: {command}")
	            stdin, stdout, stderr = ssh_client.exec_command(command)
	            command_output = stdout.read().decode()
	            command_error = stderr.read().decode()
	            print(command_output)
	            print(command_error)
	            if get_output:
	                output += command_output
	
	        ssh_client.close()
	        return output if get_output else True
	    except Exception as e:
	        print(f"SSH connection or command execution failed: {e}")
	        return False
	
	

	
