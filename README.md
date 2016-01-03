# tinc-ffsbb
tinc L2 Routingbasis fuer OSPF
Wer noch kein konfiguriertes tinc hat, kann direkt in /etc/tinc arbeiten, das dazu vorher loeschen, ansonsten TINCBASE anders setzen und bei Dateinamen aufpassen.
Wenn neue Gateways dazu kommen / Dinge geaender werden muss das lokale Repository  aktualisiert werden, damit die Keys bekannt sind.

TINCBASE=/etc/tinc/ffsbb
if [ -d "$TINCBASE" ]; then
    rm -rf $TINCBASE
fi
git clone git+ssh://git@github.com/freifunk-stuttgart/tinc-ffsbb $TINCBASE
if [ x"$TINCBASE" != x"/etc/tinc" ]; then
    rsync -rlHpogDtSvx $TINCBASE/. /etc/tinc/ffsbb
fi
if [ ! -e /etc/tinc/ffsbb/tinc.conf ]; then
    ln -s $TINCBASE/ffsbb/tinc.conf.sample /etc/tinc/ffsbb/tinc.conf
fi
if [ ! -e /etc/tinc/ffsbb/subnet-up ]; then
    ln -s $TINCBASE/ffsbb/subnet-up.sample /etc/tinc/ffsbb/subnet-up
fi
if [ ! -e /etc/tinc/ffsbb/subnet-down ]; then
    ln -s $TINCBASE/ffsbb/subnet-down.conf.sample /etc/tinc/ffsbb/subnet-down
fi
cd $TINCBASE/ffsbb
tincd -n ffsbb -K 4096
if [ x"$TINCBASE" != x"/etc/tinc" ]; then
    rsync -rlHpogDtSvx /etc/tinc/ffsbb/hosts/$HOSTNAME  $TINCBASE/ffsbb/hosts/
fi
 # Wenn man einen github Account hat, sonst den Key jemandem geben der einen hat.
git add hosts/$HOSTNAME
git commit -m "hosts/$HOSTNAME"
git push
 # Port pro GW verschieden, damit ein GW auch hinter NAT mit UDP funktioniert
cat <<EOF >>hosts/$HOSTNAME
PMTUDiscovery = yes
Digest = sha256
ClampMSS = yes
Address = $HOSTNAME.freifunk-stuttgart.de
Port = 119${HOSTNAME##gw}
EOF

cat <<'EOF' >>/etc/network/interfaces
allow-hotplug ffsbb
auto ffsbb
iface ffsbb inet manual
    tinc-net ffsbb
    tinc-mlock yes
    tinc-pidfile /var/run/tinc.ffsbb.pid
    up ip l set dev $IFACE up
EOF
