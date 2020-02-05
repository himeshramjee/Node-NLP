Steps taken to host the Node-NLP docker container on an WS LightSail instance and proxied through Nginx.
1. Launched smallest LightSail Instance with custom user script
   1. `echo '* libraries/restart-without-asking boolean true' | sudo debconf-set-selections` (get around fancy service restart prompt when installing LibSSL)
   1. or `export DEBIAN_FRONTEND=noninteractive` (I didn't test them individually, just ran both)
1. Install Docker
   1. `curl -fsSL https://get.docker.com -o get-docker.sh`
   1. `sudo apt-get install -y docker-compose`
   1. `sudo sh get-docker.sh`
   1. `sudo usermod -aG docker ubuntu` (ensure 'ubuntu' user can manage containers) 
1. Created /var/containers directory
   1. Not sure what the best practice is here but I'm for now I'm just doing a git pull so don't want config files (for example) exposed on the /var/www path.
1. Pull down the git repo
   1. `git clone https://git-codecommit.eu-west-1.amazonaws.com/v1/repos/Node-NLP`
1. Start the container
   1. `cd /var/containers/Node-NLP`
   1. `docker-compose up -d node-nlp`
   1. Verify with `curl localhost:8000/node-nlp`
1. Install Nginx (this should be dockerized too)
   1. `sudo apt-get install nginx`
1. Configure Noddy1 application (of which this node-nlp is the first sandbox project)
   1. `sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak` (only need this for quick reference)
   1. `sudo rm /etc/nginx/sites-enabled/default` (disable the default nginx welcome page)
   1. `sudo vi /etc/nginx/sites-available/node-nlp` and add the following configuration
      `server {
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
}```
1. Enable the new application
   1. `sudo ln -s /etc/nginx/sites-available/node-nlp /etc/nginx/sites-enabled/node-nlp
1. Restart nginx
   1. `sudo nginx -t` (tests the config)
   1. `sudo service nginx restart` (this is a hard reset!)

1. Enable the new application
   1. `sudo ln -s /etc/nginx/sites-available/node-nlp /etc/nginx/sites-enabled/node-nlp
1. Restart nginx
   1. `sudo nginx -t` (tests the config)
   1. `sudo service nginx restart` (this is a hard reset!)
1. Updated the projects index static page in S3 to reference this project.
