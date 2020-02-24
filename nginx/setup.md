# Setting up a web proxy (Nginx) in front of the app containers

I decided to put Nginx in front of my docker container. This document coversthe steps I took to host the Node-NLP docker container on an AWS LightSail instance and front it with an Nginx web proxy.

## Why Nginx?
I want the LightSail instance to host multiple tiny exploratory/spike/poc projects like Node-NLP. 
Each application would be hosted separtely but on the same domain. I thought I'd use AWS ALB for this but 
didn't follow through on exploring that idea due to the associated costs. I recalled choosing Nginx as the webserver for the Mr.D systems rewrite but couldn't recall details. So I needed a refresher and this projct gave me that opportunity to learn by doing again.

## Setup steps

1. Launched smallest LightSail Instance with custom user script
   1. `echo '* libraries/restart-without-asking boolean true' | sudo debconf-set-selections` (get around fancy service restart prompt when installing LibSSL)
   1. or `export DEBIAN_FRONTEND=noninteractive` (I didn't test them individually, just ran both)
1. Install Docker
   1. `curl -fsSL https://get.docker.com -o get-docker.sh`
   1. `sudo apt-get install -y docker-compose`
   1. `sudo sh get-docker.sh`
   1. `sudo usermod -aG docker ubuntu` (ensure 'ubuntu' user can manage containers) 
1. Created /var/containers directory
   * I'm not sure what the best practice is here but I'm for now I'm just doing a git pull so don't want config files (like docker-compose.yml for example) to mistakenly get exposed on the /var/www path.
1. Pull down the git repo
   1. `sudo mkdir node-nlp && sudo chown ubuntu:ubuntu node-nlp && cd node-nlp`
   1. `git clone https://github.com/himeshramjee/Node-NLP.git`
1. Start the container
   1. `docker-compose up -d node-nlp`
   1. Verify with `curl localhost:8000/node-nlp`
1. Install Nginx (this should be dockerized too)
   * `sudo apt-get install nginx`
1. Configure Noddy1 application (of which this node-nlp is the first sandbox project)
   1. `sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak`
       * This is optional. I only need this for quick reference.
   1. `sudo rm /etc/nginx/sites-enabled/default` (disable the default nginx welcome page)
   1. `sudo vi /etc/nginx/sites-available/node-nlp` and add the following configuration
        ```
        TODO: Anything not rooted under '/node-nlp' or '/api/appName' will be a problem as all requests come in on port 80 so we won't be able to proxy correctly. Consider the '/css' path for example.

        server {
                listen 80;
                listen [::]:80;

                server_name _;

                root /var/containers;

                location /node-nlp {
                        proxy_pass http://localhost:8000/;
                }

                location /api/nlp {
                        proxy_pass http://localhost:8000/api/nlp/;
                }

                location /css {
                        proxy_pass http://localhost:8000/css/;
                }

                location /js {
                        proxy_pass http://localhost:8000/js/;
                }
        }
        ```

1. Enable the new application
   * `sudo ln -s /etc/nginx/sites-available/node-nlp /etc/nginx/sites-enabled/node-nlp`
1. Restart nginx
   1. `sudo nginx -t` (tests the config)
   1. `sudo service nginx restart` (this is a hard reset!)
1. Updated the projects index static page in S3 to reference this project.
