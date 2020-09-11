# ansible-ec2-nginx


Simple Ansible task, description:
```
Цель задачи показать, как работается с инструментами.
А именно, как понимаются особенности создания инстансов, и доступа к ним.
Но самое главное, умение писать чистый, простой и понятный код,
который впоследствии будет легко сопровождать.

Имеется:
  - VPC c несколькими подсетями (subnet-aaaaaaaa, subnet-bbbbbbbb, subnet-cccccccc),
    которые *могут быть* раcположенны в разных зонах доступности (availability zones).
  - access key и secret key с админским доступом к VPC (есть vpc_id).
  - yaml файл с описанием типа инстансов:
# instances.yml 
instances: 
  - name: app 
    subnets: 
      - subnet: 'subnet-fba4719c' 
        seq_num: '01' 
      - subnet: 'subnet-fba4719c' 
        seq_num: '02' 
      - subnet: 'subnet-48965f66' 
        seq_num: '03' 
      - subnet: 'subnet-48965f66' 
        seq_num: '04' 
      - subnet: 'subnet-649d0e2e' 
        seq_num: '05'
  
  - name: web 
    subnets: 
      - subnet: 'subnet-fba4719c' 
        seq_num: '01' 
        assign_public_ip: true 
      - subnet: 'subnet-48965f66' 
        seq_num: '02' 
        assign_public_ip: true
... 
Задача:
Средствами Ansible создать идемпотентный playbook, который должен выполнить следующие требования:
  - создать инстансы (app-01, app-02, .., web-01, web-02, ..) в соответствии с указанными subnet.
  - публичные IP должны быть только у инстансов с параметром assign_public_ip: true
  - установить nginx на все инстансы.
  - динамически сконфигурировать nginx на инстансах типа "web".
    nginx должен работать как load balancer, только для "app"-ов своей AZ.
    например для web-02:
        http { 
          upstream apps { 
              server app-03; 
              server app-04; 
          }
          server { 
            listen 80;
            location / { 
              proxy_pass http://apps; 
            }
          } 
        }
- при добавлении/удалении app-ов в instances.yml и запуске плейбука, новые инстансы должны быть созданы, и конфиги "web"-ов обновлены.
```
Requirements
------------

The below requirements are needed on the host that executes this module.

- ansible 2.9.12
- python >= 2.6
- boto
- boto3
- botocor

On AWS :

- AWS-account (access key , secret key with needed access rights)
- Security Group (with opened 80 port at least) 
- private key (*.pem file)

## Run playbook

Change variables in roles/create-ec2-instances/vars/main.yml and use your own AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY and file *.pem 

```bash
export AWS_ACCESS_KEY_ID=AK..............W
export AWS_SECRET_ACCESS_KEY=Ix5.........................................7 
export AWS_REGION=us-east-2
export ANSIBLE_HOST_KEY_CHECKING=False
```

Simple run playbook:

```bash
ansible-playbook   -u ubuntu -i inventory  --private-key=../your-own-key-file.pem site.yml
```

## Get pages by web-browser  (or by curl)

Enter in address field of browser public IP of web-instanse, reload page and check changing a name of app-instance on the screen

## Change your server list

List of ec2-instances, subnets are in config file: config/instances.yml

