- hosts: all
  vars_prompt:
  - name: container_name
    prompt: "Enter Docker Container Name :"
    private: no

  tasks: 
   - name: Docker Repo
     yum_repository:
        name: docker_repo
        description: Repo for Docker 
        baseurl: https://download.docker.com/linux/centos/7/x86_64/stable
        gpgcheck: false
   - name: docker installation
     package: 
       name: "docker-ce-18.09.1-3.el7.x86_64"
       state: present
   - name: Start docker
     service:
       name: "docker"
       state: started
       enabled: yes
   - name: docker sdk
     command: pip3 install docker
   - name: pull image
     docker_image: 
       name: httpd
       source: pull
   - name: create a folder
     file: 
       path: "/root/etc/ansible/ansibleplaybook/mycode"
       state: directory
   - name: copy
     copy: 
      dest: "/var/www/html/"
      src: abhi.html
   - name: launch container
     docker_container: 
       name: "{{ container_name }}"
       state: started
       exposed_ports: "80" 
       image: httpd
       ports: 8080:80 
       volumes: /var/www/html/:/usr/local/apache2/htdocs
       command: httpd -D FOREGROUND
     register: docker_info
   - debug:
       var: docker_info.ansible_facts.docker_container.NetworkSettings.IPAddress 
   - name: "Retriveing IP dynamically and updating in inventory"
     template:
       src: "myhosts.txt"
       dest: "/root/ipadd.txt"
