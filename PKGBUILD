# Maintainer: Marcel Huber <`echo "moc tod liamg ta oofrebuhlecram" | rev`>
# Contributor: float <flo.at at gmx dot de>
# Contributor: Milan Knížek <knizek@volny.cz>

pkgname=zfs-auto-snapshot-git
pkgver=1.2.1.1.r0.g63e4438
pkgrel=1
pkgdesc="An alternative implementation of the zfs-auto-snapshot service for Linux that is compatible with zfs-linux and zfs-fuse using systemd instead of cron."
arch=(i686 x86_64)
makedepends=('git')
url="https://github.com/zfsonlinux/zfs-auto-snapshot"
license=('GPL')
source=("$pkgname"::'git+https://github.com/zfsonlinux/zfs-auto-snapshot#branch=debian')
install="$pkgname.install"
sha256sums=('SKIP')
depends=('systemd>=212')
replaces=(zfs-auto-snapshot-systemd-git)

pkgver() {
  cd "$srcdir/$pkgname"
  if GITTAG="$(git describe --abbrev=0 --tags 2>/dev/null)"; then
    local _revs_ahead_tag=$(git rev-list --count ${GITTAG}..)
    local _commit_id_short=$(git log -1 --format=%h)
    echo $(sed -e s/^${pkgname%%-git}// -e 's/^[-_/a-zA-Z]\+//' -e 's/[-_+]/./g' <<< ${GITTAG}).r${_revs_ahead_tag}.g${_commit_id_short}
  else
    echo 0.$(git rev-list --count master).g$(git log -1 --format=%h)
  fi
}

build() {
  msg 'Starting make...'
  cd "$srcdir/$pkgname"
  sed -i -e 's@/sbin@/bin@' \
         -e '/.cron./d' \
      Makefile

  ### Uncommet the following if you use zfs-fuse. There seems to be a
  ### bug in zfs-fuse, which results in snapshots being created,  but
  ### not displayed  with the "zfs list -t snapshot" command.
  ### See http://groups.google.com/group/zfs-fuse/browse_thread/thread/eab2b704d4275b8e
  #sed -i 's@SNAPPROP=.*@SNAPPROP=\"\"@' src/zfs-auto-snapshot.sh

  ### Uncomment the following 4 lines to change the snapshot name to
  ### @PREFIX_DATE_LABEL instead of @PREFIX-LABEL_DATE
  ### (this makes the automounted snapshots nicely sorted by time)
  #sed -i \
  # -e 's@SNAPNAME=\"\$opt_prefix\${opt_label:+\$opt_sep\$opt_label-\$DATE}\"@SNAPNAME=\"\$opt_prefix\${opt_label:+_$DATE\$opt_sep\$opt_label}"@' \
  # -e 's@SNAPGLOB=\"\$opt_prefix\${opt_label:+?\$opt_label}????????????????\"@SNAPGLOB=\"\$opt_prefix\${opt_label:+?????????????????\$opt_label}\"@' \
  # src/zfs-auto-snapshot.sh
  
  mkdir -p systemdfiles
  declare -a arr=("daily" "frequent" "hourly" "monthly" "weekly")
  for i in "${arr[@]}"
  do
    # write service files
    cat > systemdfiles/zfs-auto-snapshot-"$i".service <<EOF
[Unit]
Description=ZFS ${i} snapshot service

[Service]
ExecStart=$(sed -n -e 's@*/15 \* \* \* \* root @@' -e 's@exec @@' -e 's@zfs-auto-snapshot@/usr/bin/zfs-auto-snapshot@p' "$srcdir/$pkgname"/etc/zfs-auto-snapshot.cron.${i})
EOF
    
    # write timer files
    cat > systemdfiles/zfs-auto-snapshot-"$i".timer <<EOF
[Unit]
Description=ZFS ${i} snapshot timer

[Timer]
OnCalendar=$(if [[ $i == frequent ]] ; then echo "*:0/15" ; else echo $i ; fi)
Persistent=true

[Install]
WantedBy=timers.target
EOF
  done
  
  ### Uncomment the following line to change the prefix to a shorter string
  #sed -i 's@/zfs-auto-snapshot @/zfs-auto-snapshot --prefix=znap @' systemdfiles/*.service
}

package() {
  cd "$srcdir/$pkgname"
  make DESTDIR="$pkgdir" PREFIX="/usr" install
  mkdir -p $pkgdir"/usr/lib/systemd/system/"
  install -Dm644 systemdfiles/* $pkgdir"/usr/lib/systemd/system/"
}

# vim:set ts=2 sw=2 et:
