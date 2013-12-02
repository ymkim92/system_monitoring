시리얼 케이블을 이용한 원격시스템 접속
--------------------------------------

본 글에서 사용하는 원격시스템의 실제 하드웨어는 
Jetway 사에서 판매하는 통합 보드인 
`NF95A-270-LF
<http://www.jetway.com.tw/jw/ipcboard_view.asp?productid=721&proname=NF95A-270-LF>`_
을 이용한다. 국내에서는 
`carpckorea <http://www.carpckorea.co.kr/>`_
라는 업체에서 판매를 하고 있다. 

.. note:: 이거 광고 아니고요.. 제가 carpckorea에서 물건을 좀 구매하긴 했지만, 사장님 절대 모릅니다. 사장님, 가격 너무 자주 그리고 많이 올리시는 거 아닙니까? 싸게 좀 팔아주세요~~

이 보드에 우분투를 설치할 때는 모니터와 키보드를 연결해서 
하지만, 일단 설치가 끝난 경우에는 여러 대를 쌓아놓고 시리얼로 
필요한 장비에 연결하여 디버깅이나 업그레이드가 가능하다.
본 절에서는 이 방법에 대해 살펴본다.

먼저 PC와의 콘솔 연결을 위한 부팅 단계를 구분하고자 한다.

#. 부트로더로 제어권이 넘어가기 전단계 (BIOS 단계)
#. grub등의 부트로더가 시작되고 나서 로그인 프로프트 전단계 (부팅 단계)
#. 사용자와의 인터랙션이 가능한 단계 (OS 단계)

.. note:: ()안의 단계 이름들은 뭐 정해진 건 아니고... 제가 그냥 붙여 봤습니다.

http://mcchae.egloos.com/10562163 을 보면, 위 3단계별로
콘솔에서 출력을 받는 방법들이 설명되어 있다. 본 절에서는
사용자와의 인터랙션이 가능한 OS 단계에서의 출력만을 설명할 것이다. 

ubuntu 6.10 이후부터 소개된 개념으로 
`upstart <http://upstart.ubuntu.com/getting-started.html>`_ 가 있다. 
본 절에서는 upstart의 개념 자체가 중요한 것은 아니어서 자세히 
설명하지는 않겠다. upstart는 부팅시에 구동되어야 하는 기능들을
기술하는 방법으로 과거 init 가 하던 역할을 대체하고자 ubuntu를 만든
canonical에서 개발하고 밀고 있는 방법이다.

아무튼, 시리얼 콘솔 접속이 가능하도록 upstart의 스크립트를 
``/etc/init/`` 아래에 작성한다. 화일이름은 어떻게 해도 좋으나,
콘솔을 연결하기 위한 시리얼 포트의 이름을 붙인다. 
예를 들어, ttyS2 (COM3) 를 시리얼 콘솔 포트로 사용하려면 
``/etc/init/ttyS2`` 를 생성하고 아래 내용을 넣는다.

::

    start on runlevel [2345]
    stop on runleve [!2345]
    respawn
    exec /sbin/getty -L 115200 ttyS2 vt102

화일을 저장한 후 ``telinit 2`` 명령을 수행하여 
runlevel을 2로 해 주면 ttyS2 스크립트가 실행되면서 
콘솔을 이용할 준비가 끝난다.

원격시스템의 ttyS2에 케이블을 연결한 후 콘솔에서 입출력이
가능한지를 확인하라. 콘솔의 bps는 115200으로 설정해야 하며
이 값은 ``/etc/init/ttyS2`` 에서 다른 값으로 변경하여 사용이 가능하다. 

.. note:: 시리얼 콘솔이 정상적으로 동작하지 않는다면, 콘솔을 연결한 상태에서 다음 명령으로 데이터를 보내 케이블이나 시리얼 포트 자체가 정상동작하는지를 확인하라. ::

  $ stty -F /dev/ttyS2 speed 115200 cs8 -cstopb -parenb
  $ echo 'test' > /dev/ttyS2  # 콘솔로 test라는 문자열을 보낸다.  

  또는 ``minicom`` 을 설치하여 양방향 문자 전송이 가능한지 확인하는 것도 
  좋은 방법이다. 

더 읽어보기

* http://www.debuntu.org/how-to-set-up-a-serial-console-on-ubuntu/
* http://www.vanemery.com/Linux/Serial/serial-console.html
