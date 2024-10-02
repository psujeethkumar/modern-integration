MQ MULTI-INSTANCE SETUP
=======================

SERVER 1 :

- Verify the version of Linux is same on both servers

  cat /etc/os-release
- Verify the version of installed MQ is same on both servers

  dspmqver
- Verify the ID of the MQM user - It should be same in ther other server.

  id
  uid=994(mqm) gid=1011(mqm) groups=1011(mqm),0(root),1012(mqbrkrs)
- Verify the shared filesystem availabilty :

  cd /opt/mqm/bin/
  ./amqmfsck -wv /shared/ha-test/mq-data/
- Create quque manager

  crtmqm -md /shared/ha-test/mq-data/qmdata/ -ld /shared/ha-test/mq-data/logs/ QMGR1
- Verify the queue manager creation

  dspmqinf -o command QMGR1
  => addmqinf -s QueueManager -v Name=QMGR1 -v Directory=QMGR1 -v Prefix=/var/mqm -v DataPath=/shared/ha-test/mq-data/qmdata/QMGR1

 run the addmqinf command in the second server.

- Start the queue manager

  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Ended immediately) DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)
  strmqm -x QMGR1
  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Running) DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)
- Create a test queue

  runmqsc QMGR1
  DEFINE QLOCAL(TEST.INPUT)
- Check the Multi-instance setup :

  endmqm -s QMGR1
  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Running elsewhere) DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)

SERVER 2 :

uid=994(mqm) gid=1011(mqm) groups=1011(mqm),0(root),1012(mqbrkrs)

- Add the queue manager info of the queu manager created in server 1 :

  addmqinf -s QueueManager -v Name=QMGR1 -v Directory=QMGR1 -v Prefix=/var/mqm -v DataPath=/shared/ha-test/mq-data/qmdata/QMGR1
- Check the status of queue manager

  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Ended immediately))	DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)
- Start the queue manager on the server 1 and cre-check the status of queue manager

  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Running elsewhere) DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)
- Start the queue manager in server2 as standby

  strmqm -x QMGR1
  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Running as standby) DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)
- Check the status of queue manager after ending in the server 1 :

  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Running) DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)
- Verify the queue created in server 2:

  runmqsc QMGR1
  DISPLAY QLOCAL(TEST.INPUT)
- Add on : Stopping the standby queue manager without affecting the active queue manager (also means un-joining the standby queue manager in multi-instance setup)

  endmqm -x QMGR1
  dspmq -m QMGR1 -o all
  => QMNAME(QMGR1)                                             STATUS(Running elsewhere) DEFAULT(no) STANDBY(Permitted) INSTNAME(Installation1) INSTPATH(/opt/mqm) INSTVER(9.1.0.7)

=======================================================

ACE NODE CREATION

=======================================================

SERVER 1:

- Create an integration node with workpath location on the GFS shared folder :

  mqsicreatebroker NODE -q QMGR1 -e /shared/ha-test/ace-data/ -d defined

SERVER 2 :

- Join the node instance on the second server :

  mqsiaddbrokerinstance NODE -e /shared/ha-test/ace-data/

By this setup, NODE will be started on the server where MQ is running and will be stopped on the server where MQ is running in standby state.

DEFINE LISTENER(QMGR1.LISTENER) TRTYPE(TCP) PORT(1414) CONTROL(QMGR)

START LISTENER(QMGR1.LISTENER)
DISPLAY LSSTATUS(*) ALL
DEFINE CHANNEL(TO.QMGR1) CHLTYPE(QMGR)
setmqaut -m QMGR1 -t qmgr -p `<user>` +all
setmqaut -m QMGR1 -n '**' -t q -p `<user>` +all
