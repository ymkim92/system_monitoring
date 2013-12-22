nagios
------

모니터링이 무엇이냐고 묻는다면, 뭐라고 답할 수 있을까?
modbus와 syslog, nagios 차원에서의 답은 이런식이 아닐까 한다.

=============       ===================================================================================================================
모니터링 방법       모니터링이란
=============       ===================================================================================================================
modbus              원격시스템의 변화하는 값들을 주기적으로 관찰하여 표시하고, 필요할 경우 서버에서 원격시스템의 설정값을 변경한다.
syslog              원격시스템에서 발생하는 이벤트 메시지를 서버에서 통합관리한다.
nagios              서버에서 원격시스템에게 정의된 내용을 묻거나, 원격시스템에 정의된 이벤트가 발생할 경우 서버로 보고한다. 단순 정보 수집에 그치지 않고 미리 정해진 횟수 만큼 연속적으로 오류가 발생하면, 정해진 관리자들에게 알림 기능을 제공한다.
=============       ===================================================================================================================


..
    새로운 정보를 추가하거나 필요없는 정보를 빼고자 할 때, 
    유연성(flexibility) 차원에서는 아래와 같은 비교가 가능하다.

    =============       =========================
    모니터링 방법       유연성
    =============       =========================
    modbus              O (모니터링 대상을 추가하고 뺄 수 있지만, 변화된 레지스터의 내용을 별도로 유지해야 함. 즉, 모니터링 값의 의미를 설명하는 문서를 요구함) 
    syslog              OO (원하는 로그를 추가하고 필요없는 로그를 빼는 과정이 매우 용이함. 로그 자체에 모니터링 내용이 설명됨)
    nagios              X (모니터링 항목의 추가 및 삭제가 비교적 복잡함)
    =============       =========================

정보의 방향 차원에서는 아래표를 보라.

=============       =========================
모니터링 방법       정보의 방향
=============       =========================
modbus              양방향 (서버 주도: 서버에서 질의를 보내고 원격시스템에서 응답을 받는 형식)
syslog              단방향 (원격시스템에서 서버로 로그 메시지 전달)
nagios              양방향 (원격시스템에서 서버로 정보를 전달할 수도 있고 서버 주도로 정보를 수신하거나 원격시스템을 제어할 수도 있음)
=============       =========================


nagios는 간단한 명령어에서 스케쥴링, 웹 UI 까지 넓은 영역을 포함하고 있어
한마디로 쉽게 설명하기 어렵다.

.. note:: `Icinga <https://www.icinga.org/>`_, `Shinken <www.shinken-monitoring.org/>`_ 등 nagios 와 동일한 또는 비슷한 기능을 하는 소프트웨어 변종들이 존재한다.

nagios 설치
^^^^^^^^^^^

.. note:: 이 절은 http://askubuntu.com/questions/145518/how-do-i-install-nagios 을 참고하여 작성되었다.

ubuntu에서 nagios 설치는 간단하다.
apache는 이미 설치되어 있다는 가정하에 아래 명령을 수행한다.

::

  $ sudo apt-get install -y nagios3

명령 수행 후 몇 몇 입력창을 보게 될 것이다.  
제일 먼저 나오는 창은 메일 전송을 담당하는 postfix 에 대한 설정이다.
postfix는 nagios를 설치하고 웹화면을 구경한 후에 설치 및 설정할 수 
있으므로 이 시점에서 중요한 부분은 아니라고 할 수 있다.
"Postfix Configuration" 이 제목인 첫 창에서 OK를 클릭한 후 아래와 같이
Internet Site를 선택한다. 

.. image:: _static/nagios/install1.png

다음에 나오는 도메인 네임은 보내는 메일 주소에 사용될 @의 뒷 부분을
입력한다.

다음으로는 nagios 웹 화면에 접속할 비밀번호를 입력하는 창이 나온다.
아이디는 nagiosadmin 이며, 이 아이디와 여기서 입력한 비밀번호로
웹화면에 로그인 할 수 있다.

설치를 마친 후 브라우저에서 ``http://localhost/nagios3`` 를 입력하면
로그인 창이 나오고 아이디와 비밀번호를 입력하면 nagios 웹 화면을
볼 수 있다.

nagios는 기본적으로 localhost의 load, 현재 사용자, 디스크 공간
등을 검사하도록 설정화일을 자동으로 생성한다. nagios 웹 화면 좌측
메뉴에서 ``services`` 를 누르면 아래와 같이 기본으로 설정된 항목들을
볼 수 있다(ubuntu 13.04 환경).


.. image:: _static/nagios/install2.png
    :scale: 70%

.. note:: Disk Space와 SSH에서 발생한 에러에 대해서는 https://help.ubuntu.com/community/Nagios3 에서 Post Install Tasks를 참고하라.


이제 nagios 설치를 완료하였으므로 설정을 해야 하지만, 그전에
nagios의 기본 개념에 대한 이해를 하고 넘어가자.

nagios core에 대한 간단한 소개
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. note:: 이 절은 David Dosephen의 Building a Monitoring Infrastructure with Nagios 를 참고하여 작성되었다.

nagios의 핵심은 작은 크기의 모니터링 프로그램인 plugin 의 스케쥴링과 
알림(notification)을 위한 프레임워크라고 정의할 수 있다.
본 절에서는 nagios plugin의 원리와 구조에 대해 알아본 후, 
nagios의 스케쥴링에 대해 알아보고자 한다.

nagios plugin
"""""""""""""
plugin은 exit 코드를 반환하여 plugin의 실행결과를 nagios에게
알릴 수 있으며,
exit 코드는 아래와 같은 의미를 갖는다.

+------+----------+
| Code | Meaning  |
+======+==========+
| 0    | Ok       |
+------+----------+
| 1    | Warning  |
+------+----------+
| 2    | Critical |
+------+----------+
| 3    | Unknown  |
+------+----------+

.. note:: ls와 같은 유닉스 명령어도 nagios plugin과 동일한 방식으로 exit code를 반환하며, ``echo $?`` 명령어로 결과를 확인할 수 있다.


nagios plugin은 exit code 이외에도 문자열을 반환하여 세부 정보를 관리자에게
알릴 수 있다.
다음은 예제 plugin이며,
echo문에서 문자열을 출력하고 exit문으로 exit code를 반환한다.

.. code-block:: sh

    #!/bin/sh
    OUTPUT='ping -c5 8.8.8.8 | tail -n2'
    if [ $? -gt 0 ]
    then
        echo "CRITICAL!! $OUTPUT"
        exit 2
    else
        echo "OK! $OUTPUT"
        exit 0
    fi



nagios plugin의 역할은 다음 두가지로 나눌 수 있다.

* Host로부터 정보를 가져온다. (예, CPU 로드, index.html)
* Host의 특정 상태나 비교 결과를 exit code로 반환한다. 

이상의 내용에서 알 수 있는 바와 같이 nagios plugin은 독립적인 
명령어의 역할도 수행할 수 있으므로 테스트 목적으로 간단하게
사용해 볼 수 있다.

.. note:: nagios에서는 많은 수의 plugin을 제공하고 있다. https://www.nagios-plugins.org/ 를 참고하라.

원격지의 호스트에 대해서도 nagios를 실행할 수 있다. 이 절에서는
ssh를 이용한 원격 모니터링의 원리 설명에 집중할 것이다.
원격 호스트의 상태를 모니터링하기 위해서 ssh의 원격지 명령어 수행방법을
이용한다. 아래 명령은 원격 호스트 example.org의 
test 계정 홈 디렉토리에서 ls를 수행한
결과를 반환한다.

::
    
    $ ssh test@example.org "ls -CF"
    build/				 log/
    tmp/
    
이 명령어에서 "ls -CF" 부분을 nagios plugin으로 교체하면 ssh 문 
자체로 nagios plugin과 같은 역할을 하게 된다. 

원격호스트(example.org)에 
``/usr/local/bin/load_checker.sh`` 를 생성하고 아래 코드를
내용으로 입력하라. 시스템 부하의 값이 1를 넘어가면 Critical 오류를
발생시키는 코드이다.

.. code-block:: sh

    #!/bin/bash
    LOAD=`uptime | awk '{print $12}'`

    if (( $(bc <<< "$LOAD > 1") ))
    then
        echo "Critical! load on 'hostname' is $LOAD"
        exit 2
    else
        echo "OK! Load on 'hostname' is $LOAD"
        exit 0
    fi

다음 명령을 실행하여 실행권한을 주고 실행시켜 보자.

::

    $ sudo chmod a+x /usr/local/bin/load_checker.sh
    $ load_checker.sh
    OK! Load on 'hostname' is 0.15
    $ echo $?
    0
    
이제 아래 명령으로 원격호스트의 명령을 실행시킬 수 있다.

::
    
    $ ssh test@example.org /usr/local/bin/load_checker.sh
    OK! Load on 'hostname' is 0.13
    $ echo $?
    0
    
위의 ssh 문을 nagios의 plugin으로 만들기 위해 아래와 같은 스크립트를 
작성하여 서버에 저장한다.

.. code-block:: sh

    #!/bin/sh
    #get the ouput from the remote load_checker script
    OUTPUT=`ssh test@example.org "/usr/local/bin/load_checker.sh"`

    #get the exit code
    CODE=$?
    echo $OUTPUT
    exit $CODE

nagios 서버에 위치한 위 코드는 완벽한 nagios plugin으로 
원격호스트의 시스템 부하에 대한 출력문과 
exit 코드를 반환한다.

이 방법은 nagios에서의 원격 모니터링 원리를 잘 설명하지만,
하나의 단점이 존재한다. 서버에서 원격 시스템으로 로그인 없이
ssh 접속이 가능해야 한다. 
이에 대해서는 :ref:`remote-ssh` 을 참고하라.

nagios에서는 ssh를 이용하는 방법이외에 nagios에서 개발한
NRPE (Nagios Remote Plugin Executor)를 이용해 원격시스템의
모니터링을 수행할 수 있다. 자세한 방법은 각자 알아보시고,
여기서는 이 정도로 마무리하고자 한다.

스케쥴링
""""""""


nagios 설정
^^^^^^^^^^^


email notification
^^^^^^^^^^^^^^^^^^
