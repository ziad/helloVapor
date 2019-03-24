# Vapor App on Ubuntu Server
install swift on ubuntu server and run the app

Thanks to [brian](https://www.brianadvent.com/) for creating this [video](https://www.youtube.com/watch?v=B7y_8vMg5Hw). This is a summary on installing swift and vapor on ubuntu server.

## I. Install Swift and Vapor

### 1. Login and create a new user on your ubuntu server
<pre><code>ssh root@my.ip.addree.com
adduser ziad
usermod -aG sudo ziad
</code></pre>

### 2. Logout from the server and login as the new user
<pre><code>logout
ssh ziad@my.ip.address.com
</code></pre>

### 3. Update server using advanced packaging tool
<pre><code>sudo apt-get update</code></pre>

### 4. Install some packages
<pre><code>sudo apt-get install clang libicu-dev libpython2.7</code></pre>

### 5. Install swift
check the latest version on [swift.org](https://swift.org/download/)
<pre><code>wget https://swift.org/builds/swift-4.2.3-release/ubuntu1804/swift-4.2.3-RELEASE/swift-4.2.3-RELEASE-ubuntu18.04.tar.gz
tar xzf swift-4.2.3-RELEASE-ubuntu18.04.tar.gz
</code></pre>

### 6. Move the extracted folder and rm the tar file
<pre><code>sudo mv swift-4.2.3-RELEASE-ubuntu18.04.tar.gz /usr/share/swift
sudo rm swift-4.2.3-RELEASE-ubuntu18.04.tar.gz
</code></pre>

### 7. Set up environment variables
<pre><code>echo "export PATH=/usr/share/swift/usr/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
</code></pre>

#### 7.1 Check swift version
<pre><code>swift --version</code></pre>

### 8. Get Vapor apt packages
<pre><code>eval "$(curl -sL https://apt.vapor.sh)"</code></pre>

### 9. Install vapor
<pre><code>sudo apt-get install swift vapor</code></pre>

DONE.
### 10. Test demo app
<pre><code>vapor new demo1
cd demo1
vapor build
vapor run serve
</code></pre>

## II. NGINX

### 11. Stop apache and install nginx
<pre><code>sudo service apache2 stop
sudo apt-get install nginx
</code></pre>

#### 11-1 Check if nginx is runing
<pre><code>systemctl status nginx</code></pre>

### 12. Nginx default configuration
Change configuration to forward calls to port 8080 (use nginx as a proxy)

<pre><code>cd /etc/nginx/site-available/
sudo vi default
</code></pre>

add this after "server_name _;"
<pre><code>try_files $uri @proxy;
location @proxy {
    proxy_pass
    http://localhost:8080;
    proxy_pass_header Server;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass_header Server;
}
</code></pre>
than save and exit the file

#### 12.1 Restart nginx
<pre><code>sudo systemctl restart nginx</code></pre>

## III. Depoly

### 13. Depoly from my local computer to my ubuntu server
1. create a repo on github, gibLab or bitbucket
2. create a vapor app on my mac localy
3. git init, git add and git commit my files/changes
4. add remote with:<code><pre>git remote add origin https://my-git-repo.com/demo2.git</code></pre>
5. check with:<code><pre>git remote -v</code></pre>
6. push to remote repo:<code><pre>git push -u origin master</code></pre>
7. got the the server and clone the repo:<code><pre>git clone https://my-git-repo.com/demo2.git</code></pre>
8. build the app and start it:<pre><code>cd demo2
vapor build -release 
</code></pre>
DONE

## IV. Auto Start

### 14. Install and config supervisor to auto run/restart the app
[supervisord.org](http://supervisord.org/) or see this [How To Install and Manage Supervisor on Ubuntu and Debian VPS](https://www.digitalocean.com/community/tutorials/how-to-install-and-manage-supervisor-on-ubuntu-and-debian-vps)
<pre><code>sudo apt-get install supervisor</code></pre>

### 15. Create Supervisor config file
<pre><code>sudo vi /etc/superviser/conf.d/demo2.conf</code></pre>

#### 15.1 Add this content to the file and save + exit
<pre><code>[program:YOUR_PROGRAM_NAME]
command=/home/YOUR_USERNAME/YOUR_PROJECT/.build/release/Run serve --
env=production
directory=/home/YOUR_USERNAME/YOUR_PROJECT/
autorestart=true
user=YOUR_USERNAME
stdout_logfile=/var/log/supervisor/%(program_name)-stdout.log
stderr_logfile=/var/log/supervisor/%(program_name)-stderr.log
</code></pre>

##### 15.1.1 For my example this will be:
<pre><code>[program:demo2]
command=/home/ziad/demo2/.build/release/Run serve --env=production
directory=/home/ziad/demo2/
autorestart=true
user=ziad
stdout_logfile=/var/log/supervisor/%(program_name)-stdout.log
stderr_logfile=/var/log/supervisor/%(program_name)-stderr.log
</code></pre>
### 16. Reread the config file
<pre><code>sudo supervisorctl reread
sudo supervisorctl add demo2
sudo supervisor start demo2
</code></pre>

#### 16.1 If there is a problem with superviser try to check with
<pre><code>sudo service supervisor reload</code></pre>

### FINISH 
