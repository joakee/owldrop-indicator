# owldrop-indicator: a desktop-agnostic AirDrop tray menu, driving the omdrop command.
#
# Build from a checkout of this repository:  makepkg -si
pkgname=owldrop-indicator
pkgver=0.1.0
pkgrel=1
pkgdesc="Desktop-agnostic AirDrop tray indicator: receive, send and get notified from any panel, through omdrop"
arch=('any')
license=('MIT')
depends=('python' 'python-gobject' 'gtk3' 'libdbusmenu-gtk3' 'libnotify' 'systemd')
optdepends=('omdrop-owl: AirDrop radio backend for non-Apple Wi-Fi cards'
            'xfce4-notifyd: notifications on XFCE')

package() {
  install -Dm755 "${startdir}/owldrop-indicator" "${pkgdir}/usr/bin/owldrop-indicator"
  local i
  for i in "${startdir}"/icons/*.svg; do
    install -Dm644 "$i" "${pkgdir}/usr/share/icons/hicolor/scalable/status/${i##*/}"
  done
  install -Dm644 "${startdir}/owldrop-indicator.desktop" \
    "${pkgdir}/usr/share/applications/owldrop-indicator.desktop"
  install -Dm644 "${startdir}/owldrop-indicator.desktop" \
    "${pkgdir}/etc/xdg/autostart/owldrop-indicator.desktop"
  install -Dm644 "${startdir}/README.md" "${pkgdir}/usr/share/doc/${pkgname}/README.md"
  install -Dm644 "${startdir}/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
