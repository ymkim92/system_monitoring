로그
----

모니터링 시스템에서 로그 보다 더 중요한 것이 있을까? 나는 없다고 생각한다.
자신만의 로그를 만들어 사용할 수도 있겠지만, 
사실상의 로그 표준으로 사용되고 있는 syslog 그 중에서도 ubuntu에서
기본으로 사용하는 rsyslog를 이용하여 로그를 출력하고자 한다.
편의상 rsyslog를 syslog로 부르도록 하고 꼭 rsyslog 로 구분하여
나타낼 필요가 있을 때만 rsyslog 를 사용할 것이다.

syslog를 이용하면, 원격시스템 내부에서는 물론이고 로그의 내용을
서버로 전송하여 화일이나 DB에 저장하여 볼 수 있다. 단순한 기능 같지만,
원격시스템 모니터링에 있어서 가장 중요한 기능 중 하나라고 할 수 있다.
빈번하게 변화하는 값들은 modbus 프로토콜을 이용하여 모니터링하고, 
중요한 이벤트나 디버깅 정보등은 syslog로 관리하면 편리하다.
syslog는 특정 이벤트가 발생하는 시점들을 관리할 수도 있고, 시스템에 문제가
발생했을 때 원인을 분석하는 디버깅 용도로도 매우 유용하다.

syslog 출력의 예
^^^^^^^^^^^^^^^^

다음은 ``/var/log/syslog`` 의 내용 일부이다. syslog가 설치된 시스템에서는
기본적으로 이 화일에 시스템에서 발생하는 로그들을 출력한다. 

.. note:: apache와 같은 프로그램은 별도의 로그 파일에 로그를 기록한다. ubuntu의 경우 ``/var/log/apache2/`` 아래에 access log과 error log를 별도로 저장한다.

::

    Dec  5 10:52:25 ymkim-AO756 anacron[1058]: Job cron.daily terminated
    Dec  5 10:52:25 ymkim-AO756 anacron[1058]: Normal exit (1 job run)

    ^^^^^^^^^^^^^^^ ^^^^^^^^^^^ ^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^
    DATE   TIME     hostname    process name   message (log content) 

위의 예에서 보인바와 같이 syslog의 출력은 날짜와 시간으로 시작한다. 
이후 호스트 이름과 프로세스 이름을 출력하며, 여기까지는 
syslog에서 자동으로 출력해 주는 부분이다. 콜론(:) 이후의 내용이 
로그를 찍는 이유를 설명하는 메시지 부분이다.

log level
^^^^^^^^^
본격적으로 syslog를 설명하기 앞서서 기본적인 내용를 하나 공부하고
넘어가야 한다. 바로 메시지의 특성을 정의하는
facility와 severity에 대한 내용이다.
facility는 메시지를 발생시킨 프로그램의 타입을 나타내는 값이며,
severity는 메시지의 성격 또는 중요도를 나타낸다.
syslog에서는 이 값에 따라 로그 메시지를 어느 화일에 기록할지,
누구에게 이 사실을 알릴 것인지를 결정한다.

- facility : auth, authpriv, daemon, cron, ftp, lpr, kern, mail, news, syslog, user, uucp, local0, ... , local7  

+-----------------+----------+------------------------------------------+
| Facility Number | Keyword  | Facility Description                     |
+=================+==========+==========================================+
| 0               | kern     | kernel messages                          |
+-----------------+----------+------------------------------------------+
| 1               | user     | user-level messages                      |
+-----------------+----------+------------------------------------------+
| 2               | mail     | mail system                              |
+-----------------+----------+------------------------------------------+
| 3               | daemon   | system daemons                           |
+-----------------+----------+------------------------------------------+
| 4               | auth     | security/authorization messages          |
+-----------------+----------+------------------------------------------+
| 5               | syslog   | messages generated internally by syslogd |
+-----------------+----------+------------------------------------------+
| 6               | lpr      | line printer subsystem                   |
+-----------------+----------+------------------------------------------+
| 7               | news     | network news subsystem                   |
+-----------------+----------+------------------------------------------+
| 8               | uucp     | UUCP subsystem                           |
+-----------------+----------+------------------------------------------+
| 9               | clock    | daemon                                   |
+-----------------+----------+------------------------------------------+
| 10              | authpriv | security/authorization messages          |
+-----------------+----------+------------------------------------------+
| 11              | ftp      | FTP daemon                               |
+-----------------+----------+------------------------------------------+
| 12              | .        | NTP subsystem                            |
+-----------------+----------+------------------------------------------+
| 13              | .        | log audit                                |
+-----------------+----------+------------------------------------------+
| 14              | .        | log alert                                |
+-----------------+----------+------------------------------------------+
| 15              | cron     | clock daemon                             |
+-----------------+----------+------------------------------------------+
| 16              | local0   | local use 0 (local0)                     |
+-----------------+----------+------------------------------------------+
| 17              | local1   | local use 1 (local1)                     |
+-----------------+----------+------------------------------------------+
| 18              | local2   | local use 2 (local2)                     |
+-----------------+----------+------------------------------------------+
| 19              | local3   | local use 3 (local3)                     |
+-----------------+----------+------------------------------------------+
| 20              | local4   | local use 4 (local4)                     |
+-----------------+----------+------------------------------------------+
| 21              | local5   | local use 5 (local5)                     |
+-----------------+----------+------------------------------------------+
| 22              | local6   | local use 6 (local6)                     |
+-----------------+----------+------------------------------------------+
| 23              | local7   | local use 7 (local7)                     |
+-----------------+----------+------------------------------------------+

- severity : Emergency, Alert, Critical, Error, Warning, Notice, Info or Debug 

+------+---------------+----------------+-----------------------------------+
| Code | Severity      | Keyword        | Description                       |
+======+===============+================+===================================+
| 0    | Emergency     | emerg (panic)  | System is unusable.               |
+------+---------------+----------------+-----------------------------------+
| 1    | Alert         | alert          | Action must be taken immediately. |
+------+---------------+----------------+-----------------------------------+
| 2    | Critical      | crit           | Critical conditions.              |
+------+---------------+----------------+-----------------------------------+
| 3    | Error         | err (error)    | Error conditions.                 |
+------+---------------+----------------+-----------------------------------+
| 4    | Warning       | warning (warn) | Warning conditions.               |
+------+---------------+----------------+-----------------------------------+
| 5    | Notice        | notice         | Normal but significant condition. |
+------+---------------+----------------+-----------------------------------+
| 6    | Informational | info           | Informational messages.           |
+------+---------------+----------------+-----------------------------------+
| 7    | Debug         | debug          | Debug-level messages.             |
+------+---------------+----------------+-----------------------------------+

facility와 severity에 따라 어떤 화일에 로그를 쓸지에 대해, 
rsyslog의 경우 /etc/rsyslog.d/50-default.conf 에 정의되어 있다.



쉘 명령어로 syslog 출력하기
^^^^^^^^^^^^^^^^^^^^^^^^^^^

시스템에서 발생하는 로그가 아닌 직접 생성한 로그를 만드는 
가장 쉬운 방법은 ``logger`` 명령어를 이용하는 것이다.
``man logger`` 에서 모든 옵션에 대한 설명을 찾을 수 있으며, 여기서는
몇 가지 예만을 보도록 한다.

::

    $ logger log test..
    $ tail -f /var/log/syslog

tail 명령어 출력 결과의 마지막에서 아래와 같은 내용을 볼 수 있을 
것이다.

:: 

    Dec  5 15:30:37 ymkim-AO756 ymkim: log test..
    
log level을 입력으로 넣기 위해서는 -p 옵션을 사용한다.
-p 옵션 뒤에 ``facility.severity`` 를 입력한다. 
-p 옵션을 사용하기 전에 local0 facility의 새로운 출력화일을 지정하기 위해
``/etc/rsyslog.d/50-default.conf`` 의
제일 아래 줄에 다음 내용를 추가하자.

::

    local0.*	/var/log/test.log

.. note:: rsyslog의 메인 설정화일은 /etc/rsyslog.conf 이다. 이 화일에서 /etc/rsyslog.d/50-default.conf 을 불러와 추가적인 내용을 설정한다. 설정화일에 대한 자세한 내용은 man rsyslog.conf 를 확인하라.

변경내용을 저장한 후 아래 명령으로 rsyslog를 재시작한다.

::

    $ sudo restart rsyslog


이제 아래 명령을 실행시키면 ``/var/log/test.log`` 와
``/var/log/syslog`` 에서 입력한 로그 메시지를 확인할 수 있다.

::

    $ logger -p local0.info log test 2..

``/var/log/syslog`` 로는 로그를 쓰지 않도록 하기 위해 
``/etc/rsyslog.d/50-default.conf``
설정화일에서 아래 내용을 찾아서,

::

    *.*;auth,authpriv.none      -/var/log/syslog

다음과 같이 local0.none을 추가하라

::

    *.*;auth,authpriv.none,local0.none      -/var/log/syslog

이 줄의 의미는 모든 로그(\*.\*)를 /var/log/syslog에 기록하지만, 세미콜론(;)
이후의 facility들인 auth, authpriv, local0 은 제외(none)하라는 것이다.
화일이름 앞의 ``-`` 은 로그를 화일에 바로 쓰지 말고 메모리에 
로그를 가지고 있다가 디스크에 입출력 여유가 있을 경우 쓰라는 의미이다
(http://shallowsky.com/blog/linux/rsyslog-conf-tutorial.html
의 Rules Section을 보라).


날짜와 시간 형식 변경
^^^^^^^^^^^^^^^^^^^^^

syslog의 기본 날짜에는 년도가 빠져있다. 어떤이는 이에 불만(?)을 가질 수 있다.
또 다른이는 좀 더 정확한 시간을 기록하고 싶어한다. 
이런 사람들의 요구를 만족하기 위한 방법을 알아보자.

[20130227-1130]
rsyslog의 기본 timestamp에 년도 추가하기?
- http://www.rsyslog.com/using-a-different-log-format-for-all-files/
rsyslog에 milli second 추가하기

==>
/etc/rsyslog.d/50-default.conf 화일을 아래와 같이 수정

맨 위에 아래 줄 추가
$template myFormat,"%TIMESTAMP:::date-pgsql% %HOSTNAME% %syslogtag%%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"

변경된 포맷을 적용할 화일을 아래와 같이 선택
local0.*            /var/log/mb_serial.log;myFormat

아래와 같이 변경된 포맷으로 출력
Feb 27 12:03:07 ymkim-AO756 ymkim: log test..1
2013-02-27 12:38:33 ymkim-AO756 ymkim: log test..3


아래를 사용하면
$template myFormat,"%TIMESTAMP:::date-pgsql%.%timereported:1:3:date-subseconds% %HOSTNAME% %syslogtag%%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"
이런 결과
2013-02-27 13:55:46.428 ymkim-AO756 ymkim: log test..3

DateFormat  New format, additional parameter is needed. See below.
mysql   format as mysql date
pgsql   format as pgsql date
rfc3164 format as RFC 3164 date
rfc3164-buggyday    similar to date-rfc3164, but emulates a common coding error: RFC 3164 demands that a space is written for single-digit days. With this option, a zero is written instead. This format seems to be used by syslog-ng and the date-rfc3164-buggyday option can be used in migration scenarios where otherwise lots of scripts would need to be adjusted. It is recommended not to use this option when forwarding to remote hosts - they may treat the date as invalid (especially when parsing strictly according to RFC 3164).
rfc3339 format as RFC 3339 date
unixtimestamp   format as unix timestamp (seconds since epoch)
subseconds  just the subseconds of a timestamp (always 0 for a low precision timestamp)




hostname 설정
^^^^^^^^^^^^^
ubuntu 설치시 사용자 아이디와 컴퓨터 이름을 조합해 자동으로 hostname을
만들어 준다. 참 편리한 기능이다. 하지만, 동일한 장비에 ubuntu를 설치하면
모든 장비의 hostname이 같아지는 현상이 발생한다.
syslog는 원격으로 로그를 보내 통합하여 관리하는 기능이 있으므로
hostname을 다르게 설정해 주는 것이 중요하다.

- /etc/hostname의 값을 변경
  $ hostname -F /etc/hostname // rebooting 없이 재 로그인만으로 확인 가능
- /etc/hosts 도 함께 변경해 주어야 함



C 코드에서 syslog 출력하기
^^^^^^^^^^^^^^^^^^^^^^^^^^

logrotate로 최신 로그만 남기기
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^




서버로 syslog 출력 보내기
^^^^^^^^^^^^^^^^^^^^^^^^^

웹에서 syslog 결과보기 (Log Analyzer)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

http://loganalyzer.adiscon.com/





