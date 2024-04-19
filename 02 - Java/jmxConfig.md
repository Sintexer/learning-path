# JMX java opts config for app in docker to attach VisualVM

    -Dcom.sun.management.jmxremote=true
    -Dcom.sun.management.jmxremote.port=9010
    -Dcom.sun.management.jmxremote.local.only=false
    -Dcom.sun.management.jmxremote.authenticate=false
    -Dcom.sun.management.jmxremote.rmi.port=9010
    -Dcom.sun.management.jmxremote.ssl=false
    -Djava.rmi.server.hostname=localhost
