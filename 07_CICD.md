# CI/CD

[virtualbox 다운로드](https://www.virtualbox.org/)

[vagrant 다운로드](https://www.vagrantup.com/)

> virtualbox를 사용하기 위해서는 docker를 중단하고 실행한다.
>
> hypervisor off 확인 (관리자 명령 cmd)
>
> ```
> C:\Windows\system32>bcdedit
> ```
>
> ```
> C:\Windows\system32>bcdedit /set hypervisorlaunchtype off
> ```
>
> Vagrantfile에 public_network -> private_network로 변경한 후 cmd창에 vagrant halt한 다음 vagrant up 해준다.

![image](https://user-images.githubusercontent.com/42603919/82625359-ddf59580-9c1f-11ea-9050-924b93ca08c7.png)



```powershell
C:\Users\HPE>vagrant plugin install vagrant-vbguest
```



##### 다음에 해당하는 디렉토리로 이동한 후 다음 명령어를 실행.

```powershell
C:\Users\HPE\work\vagrant>vagrant up
```



##### 설치 완료 후 jenkins-server에 접속해보기

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh jenkins-server
[vagrant@jenkins-server ~]$
```



##### jenkins-server에 java 설치하기

```powershell
[vagrant@jenkins-server ~]$sudo yum -y install java-1.8*
```

```powershell
[vagrant@jenkins-server ~]$find /usr/lib/jvm/java-1.8* | head -n 3
/usr/lib/jvm/java-1.8.0
/usr/lib/jvm/java-1.8.0-openjdk
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
```

- bash_profile vi 편집기에 들어가기

```powershell
[vagrant@jenkins-server ~]$ vi ~/.bash_profile
```

- bash_profile vi 편집기에서 작업

```
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-<Java version which seen in the above output>
export JAVA_HOME
PATH=$PATH:$JAVA_HOME (PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME)

shift + : => wq!

[vagrant@jenkins-server ~]$ exit

C:\Users\HPE\work\vagrant> vagrant ssh jenkins-server (다시 로그인)
```





##### jenkins 설치하기 (jenkins-server에서)

```powershell
[vagrant@jenkins-server ~]$ sudo yum -y install wget
```

```powershell
[vagrant@jenkins-server ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

```powershell
[vagrant@jenkins-server ~]$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

```powershell
[vagrant@jenkins-server ~]$ sudo yum -y install jenkins
```



##### jenkins 시작

```powershell
[vagrant@jenkins-server ~]$ sudo service jenkins start
```

```powershell
[vagrant@jenkins-server ~]$ sudo chkconfig jenkins on
```

```powershell
[vagrant@jenkins-server ~]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```



<img src="C:\Users\HPE\AppData\Roaming\Typora\typora-user-images\image-20200522140442291.png" alt="image-20200522140442291"/>



![image](https://user-images.githubusercontent.com/42603919/82633942-7c412580-9c37-11ea-9a0a-99333c6b50bf.png)



---



##### 설치 완료 후 tomcat-server에 접속해보기

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh tomcat-server
[vagrant@tomcat-server ~]$
```



##### tomcat-server에 java 설치하기

```powershell
[vagrant@tomcat-server ~]$sudo yum -y install java-1.8*
```

```powershell
[vagrant@tomcat-server ~]$find /usr/lib/jvm/java-1.8* | head -n 3
/usr/lib/jvm/java-1.8.0
/usr/lib/jvm/java-1.8.0-openjdk
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
```

- bash_profile vi 편집기에 들어가기

```powershell
[vagrant@tomcat-server ~]$ vi ~/.bash_profile
```

- bash_profile vi 편집기에서 작업

```
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-<Java version which seen in the above output>
export JAVA_HOME
PATH=$PATH:$JAVA_HOME (PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME)

shift + : => wq!

[vagrant@tomcat-server ~]$ exit

C:\Users\HPE\work\vagrant> vagrant ssh tomcat-server (다시 로그인)
```



##### apache-tomcat 설치하기 (tomcat-server에서)

```powershell
[vagrant@tomcat-server ~]$ sudo yum -y install wget
```

```powershell
[vagrant@tomcat-server ~]$ sudo wget http://mirror.navercorp.com/apache/tomcat/tomcat-9/v9.0.35/bin/apache-tomcat-9.0.35
.tar.gz
```

```powershell
[vagrant@tomcat-server ~]$ sudo mkdir -p /usr/local/tomcat
```

```powershell
[vagrant@tomcat-server ~]$ sudo mv apache-tomcat-9.0.35.tar.gz /usr/local/tomcat
```

```powershell
[vagrant@tomcat-server ~]$ cd /usr/local/tomcat
```

```powershell
[vagrant@tomcat-server tomcat]$ sudo tar -xvzf apache-tomcat-9.0.35.tar.gz
```



##### apache-tomcat 시작

```powershell
[vagrant@tomcat-server tomcat]$ cd apache-tomcat-9.0.35/
```

```powershell
[vagrant@tomcat-server apache-tomcat-9.0.35]$ sudo ./bin/startup.sh
```



![image](https://user-images.githubusercontent.com/42603919/82642345-b49d2f80-9c48-11ea-9bdd-902e9ce2bb50.png)



##### Tomcat Manager 설정 (403 Error 해결)

1. **tomcat-users.xml** 수정

- sudo find ./ -name tomcat-users.xml (파일 위치 찾기)

```powershell
[vagrant@tomcat-server ~]$ cd /usr/local/tomcat/apache-tomcat-9.0.35
```

```powershell
[vagrant@tomcat-server apache-tomcat-9.0.35]$ sudo vi conf/tomcat-users.xml
```

```xml
# 추가

<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-script"/>
  <role rolename="manager-status"/>
  <user username="admin" password="admin" roles="manager-gui,manager-jmx,manager-script,manager-status"/>
  <user username="deployer" password="deployer" roles="manager-script"/>
  <user username="tomcat" password="tomcat" roles="manager-gui"/>
</tomcat-users>
```



2. **context.xml** 수정

- sudo find ./ -name context.xml (파일 위치 찾기)

```powershell
[vagrant@tomcat-server apache-tomcat-9.0.35]$ sudo vi /usr/local/tomcat/apache-tomcat-9.0.35/webapps/manager/META-INF/context.xml
```

```xml
<Context antiResourceLocking="false" privileged="true" >
<!--  
         <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
```



http://localhost:28080/manager/ 로 접속하면 **Tomcat 웹 애플리케이션 매니저**가 보이게 된다.



---



##### jenkins와 tomcat server 띄우기

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh jenkins-server
[vagrant@jenkins-server ~]$
```

```powershell
C:\Users\HPE\work\vagrant>vagrant ssh tomcat-server
[vagrant@tomcat-server ~]$
```



##### Server간 접속 연동하기 (다른 host로 접속)

```powershell
[vagrant@docker-server ~]$ sudo vi /etc/ssh/sshd_config
[vagrant@jenkins-server ~]$ sudo vi /etc/ssh/sshd_config
[vagrant@tomcat-server ~]$ sudo vi /etc/ssh/sshd_config
```

![image](https://user-images.githubusercontent.com/42603919/82635605-7d745180-9c3b-11ea-88ae-64954f583238.png)

```powershell
[vagrant@docker-server ~]$ sudo systemctl restart sshd
[vagrant@jenkins-server ~]$ sudo systemctl restart sshd
[vagrant@tomcat-server ~]$ sudo systemctl restart sshd
```

- 이름으로 접속할 수 있도록 설정

```
sudo vi /etc/hosts

# 각각의 Host에 자신을 제외한 나머지를 입력한다.
192.168.56.11 jenkins-server

192.168.56.12 tomcat-server

192.168.56.13 docker-server
```



##### 업로드 작업(배포 작업)

1. github에 작업할 repository를 생성

<img src="https://user-images.githubusercontent.com/42603919/82782751-0edfff80-9e98-11ea-9c0a-08b344b1677e.png" alt="image" style="zoom:67%;" />



2. 새로운 item 만들기 (jenkins)

   - jenkins관리 -> 플러그인 설치 (**github(github, integration), maven(invoke, integration)**) -> 재시작없이 설치

   ![image](https://user-images.githubusercontent.com/42603919/82783955-8b73dd80-9e9a-11ea-8c33-cb32d13b8536.png)



#### [설정]

##### 관리자 암호 변경

<img src="https://user-images.githubusercontent.com/42603919/82634935-04c0c580-9c3a-11ea-8c9a-572dfddfddc2.png" alt="image" style="zoom:67%;" />

##### JAVA 설정

![image](https://user-images.githubusercontent.com/42603919/82636803-6a16b580-9c3e-11ea-8554-45b3021201ef.png)



##### Project 생성

![image](https://user-images.githubusercontent.com/42603919/82636956-be219a00-9c3e-11ea-9d35-a2b9b88ec513.png)

![image](https://user-images.githubusercontent.com/42603919/82637238-4c961b80-9c3f-11ea-9d42-2aa4776a5c96.png)