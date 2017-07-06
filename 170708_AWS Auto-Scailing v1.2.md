
# AWS EC2 Auto-Scaling ( 사우론 )


#### 준비물 : Domain 존재하며 Nginx가 설치되었고 잘 작동하는 EC2 Server	  
	  

> [ 개념 정리 ]
> Launch Configuration : 자동으로 생성되는 ec2 instance의 사양을 설정
> Auto Scaling Group : Launch Configuration을 사용하여 늘이고 줄이는 주체
> Load Balancer : 이용자들의 트래픽을 감지
> Policy : Instance를 늘이고 줄이는 규칙들


### health check용 파일 생성

public 폴더에 health.html 파일 생성 

```
I'm healthy
```


### Nginx 세팅

이유 : reboot 되거나 image가 새로 생성되었을 때 nginx를 자동으로 켜주기 위함

1. 서버에서 

```
$ sudo vi /etc/init.d/nginx
```

새로 만들어진 nginx 파일에 다음의 코드를 붙인다.

```
#!/bin/sh
# chkconfig: - 85 15
# description: nginx is a World Wide Web server. It is used to serve
#
# processname: nginx
# config:      /opt/nginx/conf/nginx.conf
# pidfile:     /opt/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/opt/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/opt/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

2.
```
$ chmod +x /etc/init.d/nginx
```

3.

```
$ chkconfig --list
``` 

처음에는 아예 nginx 항목이 없다. 

```
$ sudo chkconfig --add nginx
```

nginx 등록 (전부다 off 인 상태로)

```
$ sudo chkconfig nginx on
```

다시 

```
$ chkconfig --list nginx
```

2 ~ 5번이 on 인 것을 확인한다.   



### Load Balancer 

1. Load Balancing 그룹 - Load Balancer 클릭

1. Step 1 : Define Load Balancer
	> Load Balancer Name : 아무거나

1.  Step 2 : Assign Security Groups
	> Security Group 선택

1. Step 3 : Configure Security Settings

1. Step 4 : Configure Health Check

	> Ping : 주기적으로 상태를 체크
	> Ping Protocol : Ping을 보내는 방식
	> Ping Port : 80
	> Ping Path : 확인할 파일 ( ex : /health.txt ( 아무 내용이나 쓰고) )
	   /public 안에 해당 파일을 만들 것!!

	> Response Timeout : 기다리는 시간, 5
	> Interval : 주기, 30
	> Unhealthy threshold : 2, 2번 반응이 없을 경우 unhealthy
	> Healthy threshold : 10, 10번 반응하면 healthy

1. Step 5 : Add EC2 Instance 

	> Load Balancer에서 체크할 대상 선택(초기에는 선택 X)

1. Step 6: Add Tags ( Skip )

1. Step 7: Review ( 확인 )


### Launch Configuration ( 버전업 할때마다 새로 생성 )

1. EC2 - Instance - 해당 인스턴스 우클릭 - Image- Create Image - 이름 설정 - 생성

1. EC2 - IMAGES -AMIs - 생성된 이미지 확인 ( 잠시 기다려라 )
	> 해당 인스턴스는 Reboot 발생

1. EC2 - AutoScaling - Launch Configuration - Create - 파란 버튼 - (복제품을 만들 것 선택) My AMIs 탭에서 이미 복제해 둔 이미지 선택 - 크기 선택


1. 세팅 확인 및 Security Groups 세팅
	- 기존의 ec2 security group 선택

1. Pem Key 세팅



### Auto Scaling

1. Auto Scaling -  Auto Scaling Groups 선택

1. Create Auto Scaling group 버튼 클릭

1. Create Auto Scailing Group
	> 앞서 만든 Launch Configuration 선택

1. Step 1. Create Auto Scaling Group
	> Receive traffic from one or more load balancers 클릭		
	> Group name : 아무거나		
	> Sub net : 아무거나	
	> Advance Details 	
		- Load Balancing : 앞서 만든 Load Balancing 파일 선택	
		- Health Check Type : ELB

1. Step 2. Configure scaling policies
	> Use Scaling policies to adjust the capacity of this group 선택
	
	> Scale between '1 (최소)' and '5 (최대)' instances. These will be the minimum and maximum size of your group. ( 숫자는 맘대로 ) 
	
	> 1) Increase Group Size ( 늘리는 규칙 )
	- Name : 아무거나
	- Excute policy when : 오른쪽 Add new alarm 클릭 ( AWS CloudWatch Service )
		`Whenever : 'Average' of 'CPU Utilization'`
		`Is : '>=' '80' Percent`
		`For at least : '1' consecutive period(2) of '5 Minutes'`
		`Name of alarm : 아무거나`
	- Take the action :  'Add' '1' 'instance' when '80' <= CPU utilization
	- Instances need (쿨타임) : '300' seconds to warm up after each step
  
	> 2) Decrease Group Size (줄이는 규칙)
	- Name : 아무거나
	- Excute policy when : 오른쪽 Add new alarm 클릭 ( AWS CloudWatch Service )
		`Whenever : 'Average' of 'CPU Utilization'`
		`Is : '<=' '20' Percent`
		`For at least : '1' consecutive period(2) of '5 Minutes'`
		`Name of alarm : 아무거나`
	- Take the action :  'Remove' '1' 'instance' when '20' >= CPU utilization

1. Step 3. Configure Notifications ( Skip )

1. Configure Tags ( Skip )

1. Step 5. Review


### 그 이후... (관리)

1. 종료 후 스스로 Instance 생성을 확인
	> Health Check 시간을 고려하여 Load Balacers에서 Status 'InService'가 됨을 확인

1. 코드를 바꾼 후 새로 Deploy를 해야 할 때
	
	Auto Scaling  Group 하단 Details 탭 'Edit' 클릭
	> Launch Configuration : 새로운 버전 선택
	> Desired : n + M ( 기존의 것 n )
	> Min : n + M ( 추가할 것 M )

	> "추가로 M개가 생성됨 (새로운 버전으로 적용)"

	> Desired : n
	> Min : n

	> "다시 줄이면 예전 버전의 것이 우선적으로 삭제"

1. Route53
	> Name : 아무거나
	> Type : A -IPv4 address
	> Alias : Yes
	> Alias Target : Load Balancer ( 이미 만든 )
	